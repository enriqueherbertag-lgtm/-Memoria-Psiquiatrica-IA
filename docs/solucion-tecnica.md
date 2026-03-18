# Solución Técnica: Memoria Fija v2

## Descripción General del Sistema

Memoria Fija v2 es un **sistema implantable de neuromodulación subcutánea** diseñado para la detección temprana y prevención de crisis psiquiátricas, así como para permitir la comunicación directa médico-paciente durante episodios agudos.

El dispositivo se implanta a nivel subcutáneo en la región **retroauricular** (detrás de la oreja), aprovechando la proximidad al cráneo para la transmisión eficiente de estímulos piezoeléctricos y la recepción de mensajes de voz.

---

## Arquitectura del Sistema

```
[Paciente con implante retroauricular]
              ↓
[Sensores fisiológicos] → [Algoritmo de detección]
              ↓
        [Detección de crisis]
              ↓
    ┌─────────┴─────────┐
    ↓                   ↓
[Estimulación         [Notificación a
 piezoeléctrica        médico vía app]
 automática]                ↓
    ↓                   [Médico envía
    ↓                    mensaje de voz]
    ↓                   ↓
    └─────────┬─────────┘
              ↓
   [Paciente recibe estímulo
    + mensaje de voz resonado]
              ↓
   [Registro en blockchain
    + ajuste adaptativo]
```

---

## Componentes del Dispositivo

### 1. Cápsula implantable (retroauricular)
| Especificación | Valor |
|:---|:---|
| Dimensiones | 35 mm × 25 mm × 8 mm |
| Peso | 12 g |
| Material | Titanio grado médico + silicona biocompatible |
| Ubicación | Subcutáneo, región retroauricular |
| Vida útil | 5 años (batería reemplazable mediante cirugía menor) |

### 2. Sensores integrados
| Sensor | Función | Frecuencia de muestreo |
|:---|:---|:---|
| Fotopletismógrafo (PPG) | Frecuencia cardíaca, variabilidad | 100 Hz |
| Sensor de conductancia dérmica | Nivel de activación autonómica | 10 Hz |
| Acelerómetro 3 ejes | Actividad motora, agitación | 50 Hz |
| Sensor de temperatura | Temperatura corporal | 1 Hz |

### 3. Actuador piezoeléctrico
| Especificación | Valor |
|:---|:---|
| Tipo | Piezoeléctrico multicapa |
| Rango de frecuencia | 100 Hz - 8 kHz |
| Intensidad máxima | 80 dB (vía ósea) |
| Modos de operación | Estimulación calmante / Mensajes de voz |
| Resolución de modulación | 256 niveles |

### 4. Módulo de comunicación
| Especificación | Valor |
|:---|:---|
| Protocolo | Bluetooth 5.3 LE |
| Alcance | 10 metros |
| Consumo | 0.5 mW en espera, 10 mW en transmisión |
| Seguridad | Cifrado AES-256 + autenticación biométrica |

### 5. Batería
| Especificación | Valor |
|:---|:---|
| Tipo | Li-ion biocompatible |
| Capacidad | 200 mAh |
| Autonomía | 7 días por carga |
| Carga | Inducción inalámbrica (cargador externo) |
| Ciclos de vida | > 1000 cargas (5 años) |

### 6. Gateway hospitalario
| Especificación | Valor |
|:---|:---|
| Dispositivo | Tablet o PC con software dedicado |
| Conexión | Bluetooth + WiFi / Ethernet |
| Capacidad | Hasta 50 pacientes por gateway |
| Registro | Almacenamiento local + nube segura |

---

## Algoritmo de Detección de Crisis

### Variables monitoreadas
| Variable | Peso en algoritmo | Umbral típico |
|:---|:---|:---|
| Frecuencia cardíaca | 25% | > 100 lpm en reposo |
| Variabilidad cardíaca (HRV) | 20% | < 20 ms |
| Conductancia dérmica | 20% | > 5 µS |
| Actividad motora | 20% | > 3 desviaciones estándar |
| Temperatura | 5% | > 37.8°C |
| Hora del día | 5% | Patrón nocturno alterado |
| Historial del paciente | 5% | Modelo personalizado |

### Lógica de detección
```python
def detectar_crisis(sensores, modelo_personalizado):
    # Calcular puntaje de riesgo en tiempo real
    puntaje = (
        sensores.fc * 0.25 +
        (1 / sensores.hrv) * 20 +
        sensores.gsr * 0.20 +
        sensores.actividad * 0.20 +
        (sensores.temp - 37) * 5 +
        calcular_patron_temporal() * 0.05 +
        modelo_personalizado.riesgo_actual() * 0.05
    )
    
    # Comparar con umbral personalizado
    if puntaje > umbral_personalizado:
        return "CRISIS_INMINENTE"
    elif puntaje > umbral_precrisis * 0.7:
        return "PRECRISIS"
    else:
        return "ESTABLE"
```

### Entrenamiento del modelo personalizado
- Período de calibración: 30 días post-implante.
- El algoritmo aprende los patrones basales de cada paciente.
- Se ajustan umbrales según historial de crisis previas.
- Actualización continua con cada evento registrado.

---

## Modos de Intervención

### Modo 1: Estimulación automática preventiva
| Nivel | Activación | Tipo de estímulo |
|:---|:---|:---|
| Precrisis | Puntaje > umbral_precrisis | Vibración suave (1-3 kHz, 5 seg) |
| Crisis inminente | Puntaje > umbral_crisis | Estimulación modulada (4-8 kHz, 30 seg) |
| Crisis aguda | Confirmación por médico | Estimulación intensiva + mensaje de voz |

### Modo 2: Comunicación médica directa
El médico puede enviar mensajes de voz al paciente a través de la app, que son transmitidos al implante y **resonados intracranealmente** mediante el actuador piezoeléctrico.

**Características:**
- El paciente percibe la voz como interna, clara y calmante.
- No requiere auriculares ni dispositivos externos.
- El mensaje puede ser pregrabado o generado en tiempo real.
- Se activa solo en crisis o a solicitud del médico.

**Flujo de comunicación:**
1. Médico graba mensaje en app (ej: "Juan, respira profundo, ya estoy contigo").
2. App envía audio cifrado al implante vía Bluetooth.
3. Implante reproduce el audio mediante actuador piezoeléctrico.
4. El sonido se transmite por vía ósea al oído interno.
5. El paciente escucha el mensaje con claridad.

---

## Seguridad y Redundancia

| Mecanismo | Descripción |
|:---|:---|
| Límites máximos | La intensidad de estimulación no puede superar umbrales de seguridad programados por el médico. |
| Apagado automático | Si se detecta sobrecalentamiento o mal funcionamiento, el dispositivo se desactiva. |
| Fallback manual | El paciente puede desactivar temporalmente la estimulación presionando el área del implante. |
| Registro inmutable | Todas las intervenciones y mensajes quedan registrados en blockchain privada. |
| Autenticación biométrica | Solo el médico autorizado puede enviar mensajes de voz. |

---

## Validación Técnica Realizada

| Prueba | Resultado | Fecha |
|:---|:---|:---|
| Simulación CFD de flujo en tejido subcutáneo | Difusión térmica < 1°C | Mar 2026 |
| Prueba de biocompatibilidad (ISO 10993) | En curso | Abr-Jun 2026 |
| Prototipo funcional en laboratorio | Comunicación Bluetooth validada | May 2026 |
| Prueba en modelo animal | Pendiente | Jul-Sep 2026 |

---

## Referencias Técnicas

1. IEEE Std 11073-10441-2023: Dispositivos médicos implantables de baja potencia.
2. ISO 14708-1:2024: Implantes quirúrgicos activos.
3. FDA Guidance: Implantable Brain-Computer Interface Devices (2024).
4. IEC 60601-1-11:2023: Equipos electromédicos para uso doméstico.
5. IEEE Transactions on Biomedical Circuits and Systems: "Piezoelectric Bone-Conduction Communication for Implantable Devices" (2025).
