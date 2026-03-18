# Driver PPG - Sensor Óptico MAX86150

## Descripción

Este driver controla el sensor **MAX86150** de Maxim Integrated, un sensor óptico integrado que permite medir:
- Fotopletismografía (PPG) para frecuencia cardíaca
- Saturación de oxígeno (SpO₂)
- Señal de ECG (electrocardiograma) de un solo canal

El sensor se comunica vía I2C con el microcontrolador nRF5340.

---

## Especificaciones del Sensor

| Parámetro | Valor |
|:---|:---|
| Modelo | MAX86150 |
| Fabricante | Maxim Integrated |
| Interfaz | I2C (hasta 400 kHz) |
| Dirección I2C | 0x5E (por defecto) |
| Canales | 3 (rojo, IR, ECG) |
| Resolución ADC | 19 bits |
| Corriente LED | 0-50 mA (programable) |
| Frecuencia muestreo | 10-1000 Hz |
| Voltaje operación | 1.8V - 3.3V |
| Consumo | 0.7 µA (shutdown) / 10 µA (medición) |

---

## Archivos del Driver

```
ppg/
├── ppg.c          # Implementación del driver
├── ppg.h          # Cabecera con API pública
└── README.md      # Este archivo
```

---

## API Pública

### Inicialización y configuración

```c
/**
 * @brief Inicializa el sensor PPG.
 *
 * @param dev Puntero al dispositivo (obtenido de Device Tree)
 * @return 0 en éxito, negativo en error
 */
int ppg_init(const struct device *dev);

/**
 * @brief Configura los parámetros del sensor.
 *
 * @param dev Puntero al dispositivo
 * @param sample_rate Frecuencia de muestreo (Hz)
 * @param led_current Corriente LED (mA)
 * @return 0 en éxito, negativo en error
 */
int ppg_configure(const struct device *dev, uint16_t sample_rate, uint8_t led_current);

/**
 * @brief Activa el sensor para comenzar la captura.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int ppg_start(const struct device *dev);

/**
 * @brief Detiene la captura y pone el sensor en bajo consumo.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int ppg_stop(const struct device *dev);
```

### Lectura de datos

```c
/**
 * @brief Lee una muestra de los tres canales.
 *
 * @param dev Puntero al dispositivo
 * @param red Pointer para almacenar valor del canal rojo
 * @param ir Pointer para almacenar valor del canal infrarrojo
 * @param ecg Pointer para almacenar valor de ECG
 * @return 0 en éxito, negativo en error
 */
int ppg_read_sample(const struct device *dev, uint32_t *red, uint32_t *ir, uint16_t *ecg);

/**
 * @brief Lee un bloque de múltiples muestras (modo FIFO).
 *
 * @param dev Puntero al dispositivo
 * @param buffer Buffer para almacenar muestras
 * @param samples Número de muestras a leer
 * @return Número de muestras leídas, negativo en error
 */
int ppg_read_fifo(const struct device *dev, uint32_t *buffer, uint16_t samples);
```

### Procesamiento básico

```c
/**
 * @brief Calcula la frecuencia cardíaca a partir de datos PPG.
 *
 * @param ir_data Buffer con datos del canal IR
 * @param length Longitud del buffer
 * @param sample_rate Frecuencia de muestreo (Hz)
 * @return Frecuencia cardíaca estimada (bpm)
 */
uint16_t ppg_calc_heart_rate(uint32_t *ir_data, uint16_t length, uint16_t sample_rate);

/**
 * @brief Calcula la saturación de oxígeno (SpO₂).
 *
 * @param red_data Buffer con datos del canal rojo
 * @param ir_data Buffer con datos del canal IR
 * @param length Longitud del buffer
 * @return SpO₂ estimada (%)
 */
uint8_t ppg_calc_spo2(uint32_t *red_data, uint32_t *ir_data, uint16_t length);
```

---

## Configuración Device Tree

En el archivo de Device Tree (`memoria_fija_v2.dts`):

```dts
&i2c1 {
    status = "okay";
    clock-frequency = <I2C_BITRATE_FAST>;
    
    max86150: max86150@5e {
        compatible = "maxim,max86150";
        reg = <0x5e>;
        label = "PPG_SENSOR";
        led-current = <20>;        /* mA */
        sample-rate = <100>;        /* Hz */
    };
};
```

---

## Ejemplo de Uso

```c
#include <zephyr/kernel.h>
#include "ppg.h"

void ppg_thread(void)
{
    const struct device *ppg_dev;
    uint32_t red, ir;
    uint16_t ecg;
    
    // Obtener dispositivo desde Device Tree
    ppg_dev = DEVICE_DT_GET(DT_NODELABEL(max86150));
    if (!device_is_ready(ppg_dev)) {
        printk("Error: sensor PPG no disponible\n");
        return;
    }
    
    // Inicializar y configurar
    ppg_init(ppg_dev);
    ppg_configure(ppg_dev, 100, 20);
    
    // Iniciar captura
    ppg_start(ppg_dev);
    
    while (1) {
        // Leer muestra
        ppg_read_sample(ppg_dev, &red, &ir, &ecg);
        
        // Procesar (ej: enviar a algoritmo de detección)
        process_ppg_data(ir, ecg);
        
        k_sleep(K_MSEC(10));  // 100 Hz
    }
}
```

---

## Pruebas Unitarias

```c
#include <zephyr/ztest.h>
#include "ppg.h"

ZTEST(ppg_tests, test_init)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(max86150));
    zassert_true(device_is_ready(dev), "Sensor no listo");
    
    int ret = ppg_init(dev);
    zassert_equal(ret, 0, "Error en inicialización");
}

ZTEST(ppg_tests, test_read_sample)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(max86150));
    uint32_t red, ir;
    uint16_t ecg;
    
    ppg_start(dev);
    k_sleep(K_MSEC(100));
    
    int ret = ppg_read_sample(dev, &red, &ir, &ecg);
    zassert_equal(ret, 0, "Error en lectura");
    zassert_not_equal(red, 0, "Canal rojo en cero");
    zassert_not_equal(ir, 0, "Canal IR en cero");
}

ZTEST_SUITE(ppg_tests, NULL, NULL, NULL, NULL, NULL);
```

---

## Consumo de Energía

| Modo | Consumo | Descripción |
|:---|:---|:---|
| Shutdown | 0.7 µA | Sensor apagado |
| Standby | 2 µA | Reloj interno activo |
| Medición (LED off) | 10 µA | ADC activo, LED apagado |
| Medición (20 mA LED) | 15 mA | Típico, 100 Hz |
| Medición (50 mA LED) | 35 mA | Máximo, 100 Hz |

### Estrategia de bajo consumo
- Sensor en shutdown cuando no se usa.
- LED solo se activa durante la medición.
- Frecuencia de muestreo reducida en modo reposo (10 Hz vs 100 Hz).

---

## Referencias

1. Maxim Integrated. (2024). "MAX86150 Datasheet: Integrated Optical Sensor".
2. Nordic Semiconductor. (2025). "nRF5340 Product Specification".
3. Zephyr Project. (2026). "I2C Driver API Reference".
4. ISO 80601-2-61:2023. "Medical electrical equipment - Particular requirements for pulse oximeter equipment".
5. IEEE 11073-10441-2023. "Health informatics - Personal health device communication - Device specialization - Cardiovascular monitoring".
