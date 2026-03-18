# Algoritmo de Detección de Crisis - Memoria Fija v2

## Descripción General

El algoritmo de detección de crisis es el núcleo del sistema Memoria Fija v2. Procesa en tiempo real los datos de los sensores fisiológicos para identificar patrones precursores de crisis psiquiátricas y activar las intervenciones correspondientes.

---

## Arquitectura del Algoritmo

```
[Datos de sensores]
    ↓
[Preprocesamiento] (filtrado, normalización)
    ↓
[Extracción de características] (HR, GSR, actividad, temperatura)
    ↓
[Modelo de detección] (umbrales adaptativos + ML)
    ↓
[Clasificación] (estable, precrisis, crisis inminente)
    ↓
[Activación de respuesta] (estimulación, notificación, voz)
```

---

## Variables de Entrada

| Variable | Sensor | Rango típico | Frecuencia |
|:---|:---|:---|:---|
| Frecuencia cardíaca (HR) | PPG | 50-120 lpm | 1 Hz |
| Variabilidad cardíaca (HRV) | PPG | 20-80 ms | 1 Hz |
| Conductancia dérmica (GSR) | GSR | 1-25 µS | 10 Hz |
| Actividad motora | Acelerómetro | 0-100 | 50 Hz |
| Temperatura | TMP117 | 35-39°C | 1 Hz |
| Postura | Acelerómetro | 0-3 | 1 Hz |
| Hora del día | RTC | 0-24 h | 1/h |

---

## Preprocesamiento

### Filtrado de señales

```c
/**
 * @brief Aplica filtro pasa-bajo a señal PPG.
 *
 * @param input Buffer de entrada
 * @param output Buffer de salida
 * @param len Longitud del buffer
 * @param cutoff Frecuencia de corte (Hz)
 * @param sample_rate Tasa de muestreo (Hz)
 */
void filter_lowpass(float *input, float *output, uint16_t len, float cutoff, float sample_rate)
{
    float rc = 1.0 / (2 * M_PI * cutoff);
    float dt = 1.0 / sample_rate;
    float alpha = dt / (rc + dt);
    
    output[0] = input[0];
    for (int i = 1; i < len; i++) {
        output[i] = output[i-1] + alpha * (input[i] - output[i-1]);
    }
}

/**
 * @brief Detecta y elimina artefactos de movimiento.
 *
 * @param ppg_data Buffer de datos PPG
 * @param accel_data Buffer de datos de acelerómetro (simultáneo)
 * @param len Longitud del buffer
 * @return true si hay artefacto, false en caso contrario
 */
bool detect_artifact(float *ppg_data, int16_t *accel_data, uint16_t len)
{
    // Si hay movimiento brusco, los datos PPG pueden estar corruptos
    uint32_t accel_energy = 0;
    for (int i = 0; i < len; i++) {
        accel_energy += abs(accel_data[i]);
    }
    
    return accel_energy > ARTIFACT_THRESHOLD;
}
```

### Normalización

```c
/**
 * @brief Normaliza una señal a rango [0,1] usando línea base personalizada.
 *
 * @param value Valor actual
 * @param baseline Línea base del paciente (percentil 50)
 * @param max_val Valor máximo esperado (percentil 95)
 * @return Valor normalizado (0-100)
 */
uint8_t normalize_signal(float value, float baseline, float max_val)
{
    if (value <= baseline) {
        return 0;
    }
    
    if (value >= max_val) {
        return 100;
    }
    
    return (uint8_t)((value - baseline) * 100 / (max_val - baseline));
}
```

---

## Extracción de Características

### Características derivadas

| Característica | Cálculo | Ventana | Umbral típico |
|:---|:---|:---|:---|
| HR_instant | Media de 10 segundos | 10 s | > 100 lpm |
| HRV_sdnn | Desviación estándar de intervalos RR | 5 min | < 20 ms |
| GSR_level | Media de conductancia | 30 s | > 5 µS |
| GSR_peaks | Número de picos por minuto | 1 min | > 3 |
| Activity_level | Energía total del acelerómetro | 30 s | > 500 |
| Activity_variance | Varianza de actividad | 5 min | > 200 |
| Temp_deviation | Desviación de temperatura basal | 1 h | > 1°C |
| Circadian_dev | Desviación del patrón circadiano | 24 h | > 30% |

### Implementación

```c
typedef struct {
    float hr_instant;
    float hrv_sdnn;
    float gsr_level;
    uint8_t gsr_peaks;
    uint16_t activity_level;
    uint16_t activity_variance;
    float temp_deviation;
    uint8_t circadian_dev;
} Features_t;

/**
 * @brief Extrae características de los datos de sensores.
 *
 * @param sensor_data Datos crudos de sensores
 * @param features Puntero a estructura de características
 */
void extract_features(SensorData_t *sensor_data, Features_t *features)
{
    // Frecuencia cardíaca instantánea
    features->hr_instant = calculate_hr(sensor_data->ppg_buffer, PPG_WINDOW);
    
    // Variabilidad cardíaca (SDNN)
    features->hrv_sdnn = calculate_sdnn(sensor_data->rr_intervals, RR_COUNT);
    
    // Nivel de GSR
    features->gsr_level = mean(sensor_data->gsr_buffer, GSR_WINDOW);
    
    // Picos GSR (respuestas de estrés)
    features->gsr_peaks = count_gsr_peaks(sensor_data->gsr_buffer, GSR_WINDOW);
    
    // Nivel de actividad
    features->activity_level = activity_energy(sensor_data->accel_buffer, ACCEL_WINDOW);
    
    // Varianza de actividad
    features->activity_variance = activity_variance(sensor_data->accel_buffer, ACCEL_LONG_WINDOW);
    
    // Desviación de temperatura
    features->temp_deviation = sensor_data->temp - baseline_temp;
    
    // Desviación circadiana
    features->circadian_dev = circadian_deviation(sensor_data->hour);
}
```

---

## Modelo de Detección

### Puntaje de riesgo

```c
/**
 * @brief Calcula puntaje de riesgo basado en características actuales.
 *
 * @param features Características actuales
 * @param weights Pesos del modelo (personalizados)
 * @return Puntaje de riesgo (0-100)
 */
uint8_t calculate_risk_score(Features_t *features, float *weights)
{
    float score = 0.0;
    
    score += normalize_hr(features->hr_instant) * weights[0];
    score += (100 - normalize_hrv(features->hrv_sdnn)) * weights[1];
    score += normalize_gsr(features->gsr_level) * weights[2];
    score += (features->gsr_peaks * 20) * weights[3];
    score += normalize_activity(features->activity_level) * weights[4];
    score += normalize_activity_variance(features->activity_variance) * weights[5];
    score += normalize_temp_dev(features->temp_deviation) * weights[6];
    score += features->circadian_dev * weights[7];
    
    return (uint8_t)(score > 100 ? 100 : score);
}
```

### Umbrales adaptativos

```c
/**
 * @brief Actualiza umbrales adaptativos basados en historial del paciente.
 *
 * @param patient_data Historial de datos del paciente
 */
void update_adaptive_thresholds(PatientHistory_t *patient_data)
{
    // Calcular percentiles del historial reciente (últimos 30 días)
    float p95_hr = percentile(patient_data->hr_history, 95);
    float p5_hrv = percentile(patient_data->hrv_history, 5);
    float p95_gsr = percentile(patient_data->gsr_history, 95);
    float p95_activity = percentile(patient_data->activity_history, 95);
    
    // Actualizar umbrales
    thresholds.hr_high = p95_hr * 1.2;
    thresholds.hrv_low = p5_hrv * 0.8;
    thresholds.gsr_high = p95_gsr * 1.5;
    thresholds.activity_high = p95_activity * 2.0;
    
    // Recalcular pesos del modelo basados en precisión histórica
    update_model_weights(patient_data);
}
```

### Clasificación

```c
typedef enum {
    STATE_STABLE,
    STATE_PRECRISIS,
    STATE_CRISIS_IMMINENT
} PatientState_t;

/**
 * @brief Clasifica el estado del paciente basado en puntaje de riesgo.
 *
 * @param risk_score Puntaje de riesgo actual
 * @param adaptive_thresholds Umbrales adaptativos
 * @return Estado del paciente
 */
PatientState_t classify_state(uint8_t risk_score, Thresholds_t *thresholds)
{
    if (risk_score > thresholds->crisis_threshold) {
        return STATE_CRISIS_IMMINENT;
    }
    
    if (risk_score > thresholds->precrisis_threshold) {
        return STATE_PRECRISIS;
    }
    
    return STATE_STABLE;
}
```

---

## Modelo de Machine Learning (Opcional)

Para pacientes con historial extenso, se puede utilizar un modelo de ML liviano:

```c
/**
 * @brief Modelo de red neuronal simple (1 capa oculta).
 */
typedef struct {
    float input_weights[8][16];
    float hidden_bias[16];
    float hidden_weights[16][3];
    float output_bias[3];
} NNModel_t;

/**
 * @brief Evaluación del modelo de red neuronal.
 *
 * @param features Características de entrada
 * @param model Modelo entrenado
 * @return Probabilidades de cada clase [estable, precrisis, crisis]
 */
void nn_evaluate(Features_t *features, NNModel_t *model, float *output_probs)
{
    float hidden[16];
    
    // Capa oculta
    for (int i = 0; i < 16; i++) {
        hidden[i] = 0;
        for (int j = 0; j < 8; j++) {
            hidden[i] += ((float*)features)[j] * model->input_weights[j][i];
        }
        hidden[i] += model->hidden_bias[i];
        hidden[i] = relu(hidden[i]);  // Activación ReLU
    }
    
    // Capa de salida
    for (int i = 0; i < 3; i++) {
        output_probs[i] = 0;
        for (int j = 0; j < 16; j++) {
            output_probs[i] += hidden[j] * model->hidden_weights[j][i];
        }
        output_probs[i] += model->output_bias[i];
    }
    
    // Softmax
    softmax(output_probs, 3);
}
```

---

## Integración con el Sistema

### Hilo principal de detección

```c
void detection_thread(void)
{
    Features_t features;
    uint8_t risk_score;
    PatientState_t state;
    
    while (1) {
        // Obtener datos de sensores (buffer circular)
        get_sensor_data(&sensor_data);
        
        // Extraer características
        extract_features(&sensor_data, &features);
        
        // Calcular puntaje de riesgo
        risk_score = calculate_risk_score(&features, patient_weights);
        
        // Clasificar estado
        state = classify_state(risk_score, &adaptive_thresholds);
        
        // Actuar según estado
        switch (state) {
            case STATE_CRISIS_IMMINENT:
                trigger_crisis_response();
                break;
                
            case STATE_PRECRISIS:
                trigger_precrisis_alert();
                break;
                
            case STATE_STABLE:
                // Solo registro
                break;
        }
        
        // Actualizar historial
        update_patient_history(&features, state);
        
        // Recalcular umbrales cada hora
        if (time_since_last_update > 3600) {
            update_adaptive_thresholds(&patient_history);
            time_since_last_update = 0;
        }
        
        k_sleep(K_MSEC(1000));  // 1 Hz
    }
}
```

### Respuesta a crisis

```c
void trigger_crisis_response(void)
{
    // 1. Activar estimulación adaptativa
    piezo_start_stimulation(piezo_dev, PROTOCOL_CALMING);
    
    // 2. Notificar al médico vía app
    send_medical_alert(ALERT_CRISIS_IMMINENT);
    
    // 3. Registrar evento en blockchain
    log_crisis_event();
    
    // 4. Si hay médico disponible, permitir envío de voz
    if (medical_staff_available()) {
        enable_voice_channel();
    }
    
    // Mantener estimulación por 60 segundos
    k_sleep(K_SECONDS(60));
    
    // Evaluar si la crisis remitió
    if (current_risk_score > thresholds.crisis_threshold) {
        // Segunda fase de estimulación más intensa
        piezo_start_stimulation(piezo_dev, PROTOCOL_INTENSE);
        k_sleep(K_SECONDS(30));
    }
    
    piezo_stop_stimulation(piezo_dev);
}
```

---

## Validación y Pruebas

### Conjuntos de datos de prueba

| Dataset | Descripción | Uso |
|:---|:---|:---|
| Simulado | Señales sintéticas con crisis artificiales | Pruebas unitarias |
| Histórico | Datos anonimizados de pacientes con crisis reales | Validación de algoritmos |
| Piloto | Datos de estudio clínico (30 pacientes) | Ajuste fino |

### Métricas de rendimiento

| Métrica | Objetivo | Método de cálculo |
|:---|:---|:---|
| Sensibilidad | > 90% | Verdaderos positivos / (VP + FN) |
| Especificidad | > 95% | Verdaderos negativos / (VN + FP) |
| Tiempo de detección | < 60 seg | Tiempo entre inicio de crisis y detección |
| Falsos positivos | < 1 por semana | Eventos detectados sin crisis real |

---

## Referencias

1. American Psychiatric Association. (2023). "Digital phenotyping in psychiatry".
2. IEEE EMBS. (2024). "Wearable sensors for mental health monitoring".
3. Journal of Psychiatric Research. (2025). "Machine learning for psychosis prediction".
4. Nature Digital Medicine. (2024). "AI in mental health: a systematic review".
5. World Psychiatry. (2025). "Early intervention in psychosis: state of the art".
