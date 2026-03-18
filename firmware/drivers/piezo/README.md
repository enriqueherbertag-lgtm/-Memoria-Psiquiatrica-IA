# Driver Piezoeléctrico - PIEZO-455

## Descripción

Este driver controla el actuador piezoeléctrico **PIEZO-455** de Mide Technology, junto con su driver de alto voltaje **DRV2667** de Texas Instruments. En Memoria Fija v2 se utiliza para:

- **Estimulación preventiva** durante crisis inminentes
- **Transmisión de mensajes de voz** del médico al paciente
- **Retroalimentación háptica** para confirmación de acciones
- **Alarmas** al paciente (batería baja, desconexión, etc.)

El sistema completo se comunica vía I2C con el microcontrolador nRF5340.

---

## Especificaciones del Actuador

### Actuador piezoeléctrico (PIEZO-455)
| Parámetro | Valor |
|:---|:---|
| Modelo | PIEZO-455 |
| Fabricante | Mide Technology |
| Tipo | Multicapa (MLP) |
| Dimensiones | 10 mm × 10 mm × 2 mm |
| Capacidad | 20 nF |
| Rango frecuencia | 100 Hz - 8 kHz |
| Presión acústica máx | 80 dB (vía ósea) |
| THD (distorsión) | < 1% |
| Voltaje máx | ±30 V |

### Driver (DRV2667)
| Parámetro | Valor |
|:---|:---|
| Modelo | DRV2667 |
| Fabricante | Texas Instruments |
| Interfaz | I2C (hasta 400 kHz) |
| Dirección I2C | 0x59 (por defecto) |
| Voltaje salida | hasta 105 Vpp (diferencial) |
| Ganancia | 2.8, 3.6, 4.0 V/V configurable |
| THD | < 1% |
| Consumo | 2 mA (en reposo) / 25 mA (activo) |

---

## Archivos del Driver

```
piezo/
├── piezo.c          # Implementación del driver
├── piezo.h          # Cabecera con API pública
└── README.md        # Este archivo
```

---

## API Pública

### Inicialización y configuración

```c
/**
 * @brief Inicializa el sistema piezoeléctrico (actuador + driver).
 *
 * @param dev Puntero al dispositivo (obtenido de Device Tree)
 * @return 0 en éxito, negativo en error
 */
int piezo_init(const struct device *dev);

/**
 * @brief Configura los parámetros del driver.
 *
 * @param dev Puntero al dispositivo
 * @param gain Ganancia (2.8, 3.6, 4.0)
 * @param boost Voltaje de salida máx (V)
 * @return 0 en éxito, negativo en error
 */
int piezo_configure(const struct device *dev, float gain, uint8_t boost);

/**
 * @brief Activa el sistema.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int piezo_enable(const struct device *dev);

/**
 * @brief Desactiva el sistema (bajo consumo).
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int piezo_disable(const struct device *dev);
```

### Generación de señales

```c
/**
 * @brief Genera un tono continuo a frecuencia específica.
 *
 * @param dev Puntero al dispositivo
 * @param freq Frecuencia (Hz)
 * @param amplitude Amplitud (0-100)
 * @return 0 en éxito, negativo en error
 */
int piezo_play_tone(const struct device *dev, uint16_t freq, uint8_t amplitude);

/**
 * @brief Genera un tono por duración determinada.
 *
 * @param dev Puntero al dispositivo
 * @param freq Frecuencia (Hz)
 * @param amplitude Amplitud (0-100)
 * @param duration_ms Duración (ms)
 * @return 0 en éxito, negativo en error
 */
int piezo_beep(const struct device *dev, uint16_t freq, uint8_t amplitude, uint16_t duration_ms);

/**
 * @brief Reproduce una forma de onda arbitraria.
 *
 * @param dev Puntero al dispositivo
 * @param waveform Buffer con la forma de onda (8 bits)
 * @param samples Número de muestras
 * @param sample_rate Tasa de muestreo (Hz)
 * @return 0 en éxito, negativo en error
 */
int piezo_play_waveform(const struct device *dev, uint8_t *waveform, uint16_t samples, uint16_t sample_rate);

/**
 * @brief Detiene la reproducción actual.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int piezo_stop(const struct device *dev);
```

### Mensajes de voz

```c
/**
 * @brief Reproduce un mensaje de voz almacenado en memoria.
 *
 * @param dev Puntero al dispositivo
 * @param message_id Identificador del mensaje (0-15)
 * @return 0 en éxito, negativo en error
 */
int piezo_play_message(const struct device *dev, uint8_t message_id);

/**
 * @brief Almacena un mensaje de voz en memoria.
 *
 * @param dev Puntero al dispositivo
 * @param message_id Identificador del mensaje (0-15)
 * @param data Buffer con audio (PCM 8-bit, 8 kHz)
 * @param len Longitud del buffer (máx 64 KB)
 * @return 0 en éxito, negativo en error
 */
int piezo_store_message(const struct device *dev, uint8_t message_id, uint8_t *data, uint16_t len);

/**
 * @brief Reproduce un mensaje de voz recibido vía Bluetooth en tiempo real.
 *
 * @param dev Puntero al dispositivo
 * @param data Buffer con audio (PCM 8-bit, 8 kHz)
 * @param len Longitud del buffer
 * @return 0 en éxito, negativo en error
 */
int piezo_play_stream(const struct device *dev, uint8_t *data, uint16_t len);
```

### Estimulación adaptativa

```c
/**
 * @brief Inicia un protocolo de estimulación adaptativa.
 *
 * @param dev Puntero al dispositivo
 * @param protocol_id Identificador del protocolo (0-3)
 * @return 0 en éxito, negativo en error
 */
int piezo_start_stimulation(const struct device *dev, uint8_t protocol_id);

/**
 * @brief Detiene la estimulación.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int piezo_stop_stimulation(const struct device *dev);

/**
 * @brief Ajusta parámetros de estimulación en tiempo real.
 *
 * @param dev Puntero al dispositivo
 * @param intensity Intensidad (0-100)
 * @param frequency Frecuencia (Hz)
 * @return 0 en éxito, negativo en error
 */
int piezo_adjust_stimulation(const struct device *dev, uint8_t intensity, uint16_t frequency);
```

---

## Configuración Device Tree

En el archivo de Device Tree (`memoria_fija_v2.dts`):

```dts
&i2c1 {
    status = "okay";
    clock-frequency = <I2C_BITRATE_FAST>;
    
    drv2667: drv2667@59 {
        compatible = "ti,drv2667";
        reg = <0x59>;
        label = "PIEZO_DRIVER";
        gain = <3.6>;
        boost = <50>;  /* Voltaje máx (V) */
    };
};
```

---

## Ejemplo de Uso

### Estimulación preventiva

```c
#include <zephyr/kernel.h>
#include "piezo.h"

void stimulation_thread(void)
{
    const struct device *piezo_dev;
    
    // Obtener dispositivo desde Device Tree
    piezo_dev = DEVICE_DT_GET(DT_NODELABEL(drv2667));
    if (!device_is_ready(piezo_dev)) {
        printk("Error: driver piezoeléctrico no disponible\n");
        return;
    }
    
    // Inicializar y configurar
    piezo_init(piezo_dev);
    piezo_configure(piezo_dev, 3.6, 50);
    piezo_enable(piezo_dev);
    
    while (1) {
        // Esperar señal de crisis
        k_sem_take(&crisis_signal, K_FOREVER);
        
        // Iniciar estimulación adaptativa
        piezo_start_stimulation(piezo_dev, PROTOCOL_CALMING);
        
        // Mantener durante 60 segundos
        k_sleep(K_SECONDS(60));
        
        // Detener estimulación
        piezo_stop_stimulation(piezo_dev);
    }
}
```

### Mensajes de voz

```c
void voice_message_handler(const struct device *piezo_dev, uint8_t *audio_data, uint16_t len)
{
    // Reproducir mensaje de voz en tiempo real
    piezo_play_stream(piezo_dev, audio_data, len);
    
    // O alternativamente, almacenar para reproducción posterior
    // piezo_store_message(piezo_dev, 1, audio_data, len);
    // piezo_play_message(piezo_dev, 1);
}
```

### Secuencias de estimulación adaptativa

```c
// Protocolo 0: Estimulación calmante (frecuencia baja, intensidad suave)
const uint8_t calming_waveform[] = {
    128, 132, 136, 140, 144, 148, 152, 156,
    160, 156, 152, 148, 144, 140, 136, 132,
    // ... patrón sinusoidal de 8 Hz
};

// Protocolo 1: Estimulación alerta (frecuencia media)
const uint8_t alert_waveform[] = {
    128, 160, 192, 224, 255, 224, 192, 160,
    128, 96, 64, 32, 0, 32, 64, 96,
    // ... patrón de mayor amplitud
};

void stimulation_protocols_init(const struct device *dev)
{
    // Almacenar formas de onda para acceso rápido
    piezo_store_waveform(dev, PROTOCOL_CALMING, calming_waveform, sizeof(calming_waveform));
    piezo_store_waveform(dev, PROTOCOL_ALERT, alert_waveform, sizeof(alert_waveform));
}
```

---

## Pruebas Unitarias

```c
#include <zephyr/ztest.h>
#include "piezo.h"

ZTEST(piezo_tests, test_init)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(drv2667));
    zassert_true(device_is_ready(dev), "Driver no listo");
    
    int ret = piezo_init(dev);
    zassert_equal(ret, 0, "Error en inicialización");
}

ZTEST(piezo_tests, test_tone)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(drv2667));
    
    piezo_enable(dev);
    int ret = piezo_beep(dev, 1000, 50, 500);
    zassert_equal(ret, 0, "Error generando tono");
}

ZTEST(piezo_tests, test_message)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(drv2667));
    uint8_t test_msg[] = {128, 129, 130, 131, 132};  // Audio de prueba
    
    int ret = piezo_store_message(dev, 1, test_msg, sizeof(test_msg));
    zassert_equal(ret, 0, "Error almacenando mensaje");
    
    ret = piezo_play_message(dev, 1);
    zassert_equal(ret, 0, "Error reproduciendo mensaje");
}

ZTEST_SUITE(piezo_tests, NULL, NULL, NULL, NULL, NULL);
```

---

## Consumo de Energía

| Modo | Consumo | Descripción |
|:---|:---|:---|
| Shutdown | 0.1 µA | Sistema apagado |
| Standby | 2 µA | Driver en espera |
| Tono continuo (baja intensidad) | 5 mA | 50% amplitud |
| Tono continuo (máxima intensidad) | 15 mA | 100% amplitud |
| Mensaje de voz | 10-20 mA | Depende del contenido |
| Estimulación adaptativa | 8-25 mA | Variable según protocolo |

### Estrategia de bajo consumo
- Driver en shutdown cuando no se usa.
- Estimulación solo durante crisis (promedio < 0.1% del tiempo).
- Mensajes de voz cortos (< 15 segundos).
- Boost de voltaje desactivado cuando no se necesita alta amplitud.

---

## Referencias

1. Texas Instruments. (2024). "DRV2667 Datasheet: Piezo Haptic Driver with Integrated Boost Converter".
2. Mide Technology. (2025). "PIEZO-455 Product Datasheet".
3. Texas Instruments. (2025). "SLOU367: DRV2667 Evaluation Module User's Guide".
4. Nordic Semiconductor. (2025). "nRF5340 Product Specification".
5. Zephyr Project. (2026). "I2C Driver API Reference".
6. IEEE 1857.12-2024. "Standard for Immersive Audio Coding for Wearable Devices".
