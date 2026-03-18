# Algoritmo Adaptativo - Memoria Fija v2

## Descripción General

El algoritmo adaptativo de Memoria Fija v2 permite que el sistema aprenda y se ajuste a cada paciente de forma personalizada. A diferencia de los umbrales fijos, este algoritmo:

- Aprende los patrones basales de cada paciente (línea base)
- Ajusta dinámicamente los umbrales de detección según la evolución
- Optimiza los pesos del modelo de riesgo basado en eventos históricos
- Se adapta a cambios en el estado clínico (mejoría o empeoramiento)

---

## Arquitectura del Sistema Adaptativo

```
[Datos históricos del paciente]
    ↓
[Cálculo de línea base (percentiles)]
    ↓
[Ajuste de umbrales dinámicos]
    ↓
[Optimización de pesos del modelo]
    ↓
[Personalización de respuestas]
    ↓
[Feedback del médico] → [Refinamiento continuo]
```

---

## Línea Base Personalizada

### Período de calibración inicial

```c
/**
 * @brief Estructura para almacenar la línea base del paciente.
 */
typedef struct {
    float hr_baseline;          // Frecuencia cardíaca basal (percentil 50)
    float hr_p95;               // Percentil 95 de frecuencia cardíaca
    float hr_p5;                 // Percentil 5 de frecuencia cardíaca
    
    float hrv_baseline;          // Variabilidad cardíaca basal
    float hrv_p95;
    float hrv_p5;
    
    float gsr_baseline;          // Conductancia dérmica basal
    float gsr_p95;
    
    float activity_baseline;     // Actividad basal
    float activity_p95;
    
    float temp_baseline;         // Temperatura basal
    float temp_p95;
    float temp_p5;
    
    uint8_t circadian_pattern[24]; // Patrón circadiano (por hora)
} PatientBaseline_t;

/**
 * @brief Calcula la línea base después del período de calibración.
 *
 * @param patient_data Datos de los primeros 30 días
 * @param baseline Puntero a estructura de línea base
 */
void calculate_baseline(PatientHistory_t *patient_data, PatientBaseline_t *baseline)
{
    // Calcular percentiles para cada variable
    baseline->hr_baseline = percentile(patient_data->hr_history, 50);
    baseline->hr_p95 = percentile(patient_data->hr_history, 95);
    baseline->hr_p5 = percentile(patient_data->hr_history, 5);
    
    baseline->hrv_baseline = percentile(patient_data->hrv_history, 50);
    baseline->hrv_p95 = percentile(patient_data->hrv_history, 95);
    baseline->hrv_p5 = percentile(patient_data->hrv_history, 5);
    
    baseline->gsr_baseline = percentile(patient_data->gsr_history, 50);
    baseline->gsr_p95 = percentile(patient_data->gsr_history, 95);
    
    baseline->activity_baseline = percentile(patient_data->activity_history, 50);
    baseline->activity_p95 = percentile(patient_data->activity_history, 95);
    
    baseline->temp_baseline = percentile(patient_data->temp_history, 50);
    baseline->temp_p95 = percentile(patient_data->temp_history, 95);
    baseline->temp_p5 = percentile(patient_data->temp_history, 5);
    
    // Calcular patrón circadiano
    for (int hour = 0; hour < 24; hour++) {
        baseline->circadian_pattern[hour] = calculate_hourly_mean(patient_data, hour);
    }
}
```

### Normalización respecto a línea base

```c
/**
 * @brief Normaliza un valor respecto a la línea base personalizada.
 *
 * @param value Valor actual
 * @param baseline Línea base (percentil 50)
 * @param p95 Percentil 95
 * @return Valor normalizado (0-100), donde 50 es el valor basal
 */
uint8_t normalize_to_baseline(float value, float baseline, float p95)
{
    if (value <= baseline) {
        // Por debajo de la línea base
        return (uint8_t)(50 * value / baseline);
    } else {
        // Por encima de la línea base
        float range = p95 - baseline;
        if (range <= 0) return 50;
        
        float excess = value - baseline;
        uint8_t normalized = 50 + (uint8_t)(50 * excess / range);
        return (normalized > 100) ? 100 : normalized;
    }
}
```

---

## Umbrales Dinámicos

### Estructura de umbrales adaptativos

```c
typedef struct {
    // Umbrales de crisis
    uint8_t crisis_threshold;      // Puntaje para crisis inminente
    uint8_t precrisis_threshold;    // Puntaje para precrisis
    
    // Umbrales por variable (normalizados)
    uint8_t hr_crisis;
    uint8_t hr_precrisis;
    uint8_t hrv_crisis;
    uint8_t hrv_precrisis;
    uint8_t gsr_crisis;
    uint8_t gsr_precrisis;
    uint8_t activity_crisis;
    uint8_t activity_precrisis;
    
    // Factores de ajuste
    float sensitivity_factor;       // 0.8-1.2 (menos/más sensible)
    float specificity_factor;       // 0.8-1.2 (menos/más específico)
    
    // Historial de ajustes
    uint16_t adjustments_count;
    time_t last_adjustment;
} AdaptiveThresholds_t;
```

### Ajuste basado en eventos

```c
/**
 * @brief Ajusta umbrales basado en eventos recientes.
 *
 * @param thresholds Puntero a umbrales adaptativos
 * @param events Historial de eventos (crisis reales, falsos positivos)
 */
void adjust_thresholds(AdaptiveThresholds_t *thresholds, EventHistory_t *events)
{
    // Si hay falsos positivos recientes, aumentar umbrales
    if (events->false_positives_last_month > 2) {
        thresholds->crisis_threshold += 2;
        thresholds->precrisis_threshold += 1;
        thresholds->sensitivity_factor *= 0.95;  // Menos sensible
    }
    
    // Si hubo crisis no detectadas, disminuir umbrales
    if (events->missed_crises_last_month > 0) {
        thresholds->crisis_threshold -= 3;
        thresholds->precrisis_threshold -= 2;
        thresholds->sensitivity_factor *= 1.05;  // Más sensible
    }
    
    // Limitar rangos
    if (thresholds->crisis_threshold < 60) thresholds->crisis_threshold = 60;
    if (thresholds->crisis_threshold > 95) thresholds->crisis_threshold = 95;
    if (thresholds->precrisis_threshold < 40) thresholds->precrisis_threshold = 40;
    if (thresholds->precrisis_threshold > 85) thresholds->precrisis_threshold = 85;
    
    thresholds->adjustments_count++;
    thresholds->last_adjustment = time(NULL);
}
```

### Ajuste por hora del día

```c
/**
 * @brief Ajusta umbrales según la hora del día (patrón circadiano).
 *
 * @param thresholds Umbrales base
 * @param hour Hora actual (0-23)
 * @param pattern Patrón circadiano del paciente
 * @return Umbrales ajustados
 */
AdaptiveThresholds_t circadian_adjustment(AdaptiveThresholds_t thresholds, uint8_t hour, uint8_t *pattern)
{
    float factor = (float)pattern[hour] / 50.0;  // 50 = valor promedio
    
    // Durante horas de mayor actividad, umbrales más altos
    thresholds.crisis_threshold = (uint8_t)(thresholds.crisis_threshold * (0.9 + 0.2 * factor));
    thresholds.precrisis_threshold = (uint8_t)(thresholds.precrisis_threshold * (0.9 + 0.2 * factor));
    
    return thresholds;
}
```

---

## Optimización de Pesos del Modelo

### Estructura de pesos

```c
typedef struct {
    float hr_weight;          // Peso de frecuencia cardíaca (0-1)
    float hrv_weight;         // Peso de variabilidad cardíaca
    float gsr_level_weight;   // Peso de nivel GSR
    float gsr_peaks_weight;   // Peso de picos GSR
    float activity_level_weight; // Peso de nivel de actividad
    float activity_var_weight; // Peso de varianza de actividad
    float temp_weight;        // Peso de temperatura
    float circadian_weight;   // Peso de patrón circadiano
    
    // Suma total (debe ser 1.0)
    float total;
} ModelWeights_t;

/**
 * @brief Inicializa pesos con valores por defecto (basados en población general).
 *
 * @return Pesos iniciales
 */
ModelWeights_t default_weights(void)
{
    ModelWeights_t w = {
        .hr_weight = 0.20,
        .hrv_weight = 0.15,
        .gsr_level_weight = 0.15,
        .gsr_peaks_weight = 0.10,
        .activity_level_weight = 0.15,
        .activity_var_weight = 0.10,
        .temp_weight = 0.05,
        .circadian_weight = 0.10,
        .total = 1.0
    };
    return w;
}
```

### Optimización basada en precisión histórica

```c
/**
 * @brief Optimiza los pesos del modelo usando eventos históricos.
 *
 * @param weights Pesos actuales
 * @param events Historial de eventos con características
 * @return Pesos optimizados
 */
ModelWeights_t optimize_weights(ModelWeights_t weights, EventHistory_t *events)
{
    // Algoritmo de optimización simple (gradiente descendente)
    float learning_rate = 0.01;
    
    for (int i = 0; i < events->count; i++) {
        Event_t *ev = &events->events[i];
        
        // Calcular error (diferencia entre predicción y realidad)
        float predicted = calculate_risk_score(&ev->features, &weights);
        float error = (ev->was_crisis ? 100 : 0) - predicted;
        
        // Actualizar cada peso según su contribución al error
        weights.hr_weight += learning_rate * error * ev->features.hr_instant / 100;
        weights.hrv_weight += learning_rate * error * (100 - ev->features.hrv_sdnn) / 100;
        weights.gsr_level_weight += learning_rate * error * ev->features.gsr_level / 25;
        weights.gsr_peaks_weight += learning_rate * error * ev->features.gsr_peaks / 5;
        weights.activity_level_weight += learning_rate * error * ev->features.activity_level / 100;
        weights.activity_var_weight += learning_rate * error * ev->features.activity_variance / 500;
        weights.temp_weight += learning_rate * error * ev->features.temp_deviation / 2;
        weights.circadian_weight += learning_rate * error * ev->features.circadian_dev / 100;
    }
    
    // Normalizar para que suma sea 1.0
    normalize_weights(&weights);
    
    return weights;
}
```

### Personalización por tipo de crisis

```c
/**
 * @brief Identifica el tipo de crisis predominante en el paciente.
 */
typedef enum {
    CRISIS_TYPE_ANXIOUS,      // Predominio de ansiedad (alta HR, GSR)
    CRISIS_TYPE_AGITATED,     // Predominio de agitación (alta actividad)
    CRISIS_TYPE_WITHDRAWN,    // Predominio de retraimiento (baja actividad, baja HR)
    CRISIS_TYPE_MIXED
} CrisisType_t;

/**
 * @brief Ajusta pesos según el tipo de crisis del paciente.
 *
 * @param weights Pesos actuales
 * @param crisis_type Tipo de crisis predominante
 * @return Pesos ajustados
 */
ModelWeights_t adjust_for_crisis_type(ModelWeights_t weights, CrisisType_t crisis_type)
{
    switch (crisis_type) {
        case CRISIS_TYPE_ANXIOUS:
            weights.hr_weight *= 1.3;
            weights.gsr_level_weight *= 1.3;
            weights.gsr_peaks_weight *= 1.2;
            weights.activity_level_weight *= 0.8;
            break;
            
        case CRISIS_TYPE_AGITATED:
            weights.activity_level_weight *= 1.4;
            weights.activity_var_weight *= 1.3;
            weights.hr_weight *= 1.1;
            weights.gsr_level_weight *= 1.1;
            break;
            
        case CRISIS_TYPE_WITHDRAWN:
            weights.activity_level_weight *= 0.5;
            weights.activity_var_weight *= 0.5;
            weights.hr_weight *= 0.7;
            weights.temp_weight *= 1.2;
            weights.circadian_weight *= 1.3;
            break;
            
        default:
            break;
    }
    
    normalize_weights(&weights);
    return weights;
}
```

---

## Personalización de Respuestas

### Protocolos de estimulación personalizados

```c
typedef struct {
    uint8_t protocol_id;
    uint8_t intensity;          // 0-100
    uint16_t frequency;         // Hz
    uint16_t duration;          // segundos
    uint8_t waveform[256];      // Forma de onda personalizada
} StimulationProtocol_t;

/**
 * @brief Genera protocolo de estimulación personalizado.
 *
 * @param patient_id ID del paciente
 * @param crisis_type Tipo de crisis
 * @return Protocolo personalizado
 */
StimulationProtocol_t personalized_protocol(uint16_t patient_id, CrisisType_t crisis_type)
{
    StimulationProtocol_t protocol;
    
    // Cargar preferencias del paciente (guardadas en FRAM)
    PatientPreferences_t prefs = load_patient_preferences(patient_id);
    
    switch (crisis_type) {
        case CRISIS_TYPE_ANXIOUS:
            // Frecuencias bajas y suaves para ansiedad
            protocol.intensity = prefs.calming_intensity;
            protocol.frequency = prefs.calming_frequency;
            protocol.duration = 60;
            generate_calming_waveform(protocol.waveform, protocol.frequency);
            break;
            
        case CRISIS_TYPE_AGITATED:
            // Frecuencias medias para agitación
            protocol.intensity = prefs.alert_intensity;
            protocol.frequency = prefs.alert_frequency;
            protocol.duration = 45;
            generate_alert_waveform(protocol.waveform, protocol.frequency);
            break;
            
        default:
            // Protocolo estándar
            protocol.intensity = 50;
            protocol.frequency = 100;
            protocol.duration = 60;
            generate_standard_waveform(protocol.waveform);
            break;
    }
    
    return protocol;
}
```

### Mensajes de voz personalizados

```c
/**
 * @brief Selecciona mensaje de voz según preferencias del paciente.
 *
 * @param patient_id ID del paciente
 * @param crisis_type Tipo de crisis
 * @return ID del mensaje a reproducir
 */
uint8_t select_voice_message(uint16_t patient_id, CrisisType_t crisis_type)
{
    PatientPreferences_t prefs = load_patient_preferences(patient_id);
    
    switch (crisis_type) {
        case CRISIS_TYPE_ANXIOUS:
            return prefs.anxiety_message_id;
        case CRISIS_TYPE_AGITATED:
            return prefs.agitation_message_id;
        case CRISIS_TYPE_WITHDRAWN:
            return prefs.withdrawn_message_id;
        default:
            return 0;  // Mensaje genérico
    }
}
```

---

## Feedback del Médico

### Incorporación de feedback

```c
/**
 * @brief Procesa feedback del médico sobre eventos.
 *
 * @param event_id ID del evento
 * @param doctor_feedback Feedback (correcto, falso positivo, no detectado)
 */
void process_doctor_feedback(uint16_t event_id, uint8_t doctor_feedback)
{
    Event_t *event = get_event_by_id(event_id);
    
    switch (doctor_feedback) {
        case FEEDBACK_CORRECT:
            // Reforzar pesos actuales
            event->was_correct = true;
            break;
            
        case FEEDBACK_FALSE_POSITIVE:
            // Reducir sensibilidad
            adaptive_thresholds.sensitivity_factor *= 0.95;
            event->false_positive = true;
            break;
            
        case FEEDBACK_MISSED:
            // Aumentar sensibilidad
            adaptive_thresholds.sensitivity_factor *= 1.05;
            event->missed = true;
            break;
    }
    
    // Recalcular pesos con nuevo evento
    model_weights = optimize_weights(model_weights, &event_history);
    
    // Guardar en FRAM
    save_event_feedback(event_id, doctor_feedback);
}
```

### Dashboard médico

```c
/**
 * @brief Genera reporte de adaptación para el médico.
 */
typedef struct {
    float current_sensitivity;      // Sensibilidad actual
    float current_specificity;      // Especificidad actual
    uint8_t false_positives_month;  // Falsos positivos último mes
    uint8_t missed_crises_month;    // Crisis no detectadas
    
    float weights_evolution[30];    // Evolución de pesos
    uint8_t thresholds_evolution[30]; // Evolución de umbrales
    
    char recommendations[512];       // Recomendaciones de ajuste
} AdaptationReport_t;

/**
 * @brief Genera reporte de adaptación.
 *
 * @return Reporte formateado
 */
AdaptationReport_t generate_adaptation_report(void)
{
    AdaptationReport_t report;
    
    report.current_sensitivity = calculate_sensitivity(&event_history);
    report.current_specificity = calculate_specificity(&event_history);
    report.false_positives_month = count_false_positives(30);
    report.missed_crises_month = count_missed_crises(30);
    
    // Generar recomendaciones
    if (report.false_positives_month > 3) {
        sprintf(report.recommendations, 
                "Se detectaron %d falsos positivos en el último mes. "
                "Considere ajustar manualmente los umbrales o revisar "
                "la medicación del paciente.", report.false_positives_month);
    }
    
    if (report.missed_crises_month > 0) {
        sprintf(report.recommendations + strlen(report.recommendations),
                "\nHubo %d crisis no detectadas. Se recomienda aumentar "
                "la sensibilidad del dispositivo.", report.missed_crises_month);
    }
    
    return report;
}
```

---

## Validación del Sistema Adaptativo

### Pruebas de convergencia

```c
/**
 * @brief Simula aprendizaje del sistema con datos históricos.
 *
 * @param patient_data Datos históricos del paciente
 * @param iterations Número de iteraciones de entrenamiento
 * @return Error final
 */
float test_adaptive_convergence(PatientHistory_t *patient_data, uint16_t iterations)
{
    ModelWeights_t weights = default_weights();
    AdaptiveThresholds_t thresholds = default_thresholds();
    
    float initial_error = calculate_error(patient_data, &weights, &thresholds);
    
    for (int i = 0; i < iterations; i++) {
        // Simular un mes de datos
        EventHistory_t events = simulate_month(patient_data, &weights, &thresholds);
        
        // Optimizar
        weights = optimize_weights(weights, &events);
        adjust_thresholds(&thresholds, &events);
    }
    
    float final_error = calculate_error(patient_data, &weights, &thresholds);
    
    return final_error / initial_error;  // Factor de mejora
}
```

---

## Referencias

1. IEEE Transactions on Biomedical Engineering. (2024). "Adaptive Algorithms for Mental Health Monitoring".
2. Journal of Medical Systems. (2025). "Personalized Thresholds in Psychiatric Wearables".
3. Nature Digital Medicine. (2024). "Machine Learning Personalization in Digital Psychiatry".
4. Biological Psychiatry. (2025). "Circadian Patterns in Psychiatric Disorders".
5. IEEE EMBS Conference. (2024). "Real-time Adaptive Systems for Crisis Detection".
