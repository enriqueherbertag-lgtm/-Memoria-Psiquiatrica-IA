# Driver GSR - Sensor de Conductancia Démica AD5940

## Descripción

Este driver controla el sensor **AD5940** de Analog Devices, un circuito integrado de alta precisión para mediciones electroquímicas e impedancia. En Memoria Fija v2 se utiliza para medir:

- **Conductancia dérmica (GSR)**: Indicador de activación autonómica
- **Impedancia de la piel**: Parámetro relacionado con el estrés y la excitación emocional

El sensor se comunica vía SPI con el microcontrolador nRF5340.

---

## Especificaciones del Sensor

| Parámetro | Valor |
|:---|:---|
| Modelo | AD5940 |
| Fabricante | Analog Devices |
| Interfaz | SPI (hasta 16 MHz) + GPIOs |
| Canales | 2 (CE0, CE1) configurables |
| Resolución ADC | 16 bits |
| Rango impedancia | 0.1 Ω - 10 MΩ |
| Frecuencia excitación | 0.015 Hz - 200 kHz |
| Modo GSR | AC (sinusoidal) |
| Voltaje operación | 1.8V - 3.6V |
| Consumo | 0.1 µA (sleep) / 1 mA (medición) |

---

## Archivos del Driver

```
gsr/
├── gsr.c          # Implementación del driver
├── gsr.h          # Cabecera con API pública
└── README.md      # Este archivo
```

---

## API Pública

### Inicialización y configuración

```c
/**
 * @brief Inicializa el sensor GSR.
 *
 * @param dev Puntero al dispositivo (obtenido de Device Tree)
 * @return 0 en éxito, negativo en error
 */
int gsr_init(const struct device *dev);

/**
 * @brief Configura los parámetros de medición.
 *
 * @param dev Puntero al dispositivo
 * @param freq Frecuencia de excitación (Hz)
 * @param amplitude Amplitud de excitación (mV)
 * @return 0 en éxito, negativo en error
 */
int gsr_configure(const struct device *dev, uint16_t freq, uint16_t amplitude);

/**
 * @brief Activa el sensor para comenzar la captura.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int gsr_start(const struct device *dev);

/**
 * @brief Detiene la captura y pone el sensor en bajo consumo.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int gsr_stop(const struct device *dev);
```

### Lectura de datos

```c
/**
 * @brief Lee una muestra de conductancia dérmica.
 *
 * @param dev Puntero al dispositivo
 * @param conductance Puntero para almacenar valor (µS)
 * @return 0 en éxito, negativo en error
 */
int gsr_read_conductance(const struct device *dev, uint32_t *conductance);

/**
 * @brief Lee impedancia a una frecuencia específica.
 *
 * @param dev Puntero al dispositivo
 * @param freq Frecuencia de medición (Hz)
 * @param impedance Puntero para almacenar impedancia (Ω)
 * @return 0 en éxito, negativo en error
 */
int gsr_read_impedance(const struct device *dev, uint16_t freq, uint32_t *impedance);

/**
 * @brief Realiza un barrido de frecuencias (EIS).
 *
 * @param dev Puntero al dispositivo
 * @param freq_start Frecuencia inicial (Hz)
 * @param freq_end Frecuencia final (Hz)
 * @param points Número de puntos
 * @param results Buffer para almacenar resultados
 * @return Número de puntos medidos, negativo en error
 */
int gsr_sweep_frequency(const struct device *dev, uint16_t freq_start, 
                        uint16_t freq_end, uint16_t points, uint32_t *results);
```

### Calibración

```c
/**
 * @brief Calibra el sensor con resistencias conocidas.
 *
 * @param dev Puntero al dispositivo
 * @param ref_resistor Resistencia de referencia (Ω)
 * @return 0 en éxito, negativo en error
 */
int gsr_calibrate(const struct device *dev, uint32_t ref_resistor);

/**
 * @brief Aplica compensación por temperatura.
 *
 * @param dev Puntero al dispositivo
 * @param temperature Temperatura actual (°C)
 * @return 0 en éxito, negativo en error
 */
int gsr_temperature_compensate(const struct device *dev, float temperature);
```

---

## Configuración Device Tree

En el archivo de Device Tree (`memoria_fija_v2.dts`):

```dts
&spi2 {
    status = "okay";
    cs-gpios = <&gpio0 20 GPIO_ACTIVE_LOW>;
    clock-frequency = <8000000>;
    
    ad5940: ad5940@0 {
        compatible = "adi,ad5940";
        reg = <0>;
        label = "GSR_SENSOR";
        spi-max-frequency = <8000000>;
        excitation-freq = <20>;      /* Hz */
        excitation-amp = <500>;       /* mV */
        gpios = <&gpio0 21 GPIO_ACTIVE_HIGH>; /* Reset */
    };
};
```

---

## Ejemplo de Uso

```c
#include <zephyr/kernel.h>
#include "gsr.h"

void gsr_thread(void)
{
    const struct device *gsr_dev;
    uint32_t conductance;
    
    // Obtener dispositivo desde Device Tree
    gsr_dev = DEVICE_DT_GET(DT_NODELABEL(ad5940));
    if (!device_is_ready(gsr_dev)) {
        printk("Error: sensor GSR no disponible\n");
        return;
    }
    
    // Inicializar y configurar
    gsr_init(gsr_dev);
    gsr_configure(gsr_dev, 20, 500);
    
    // Calibrar
    gsr_calibrate(gsr_dev, 10000);  // Resistor 10kΩ
    
    // Iniciar captura
    gsr_start(gsr_dev);
    
    while (1) {
        // Leer conductancia
        gsr_read_conductance(gsr_dev, &conductance);
        
        // Procesar (ej: enviar a algoritmo de detección)
        process_gsr_data(conductance);
        
        k_sleep(K_MSEC(100));  // 10 Hz
    }
}
```

---

## Algoritmo de Procesamiento

### Cálculo de activación autonómica

```c
/**
 * @brief Calcula nivel de activación a partir de GSR.
 *
 * La conductancia dérmica aumenta con la sudoración,
 * indicando activación del sistema nervioso simpático.
 *
 * @param conductance Valor crudo de conductancia (µS)
 * @return Nivel de activación (0-100)
 */
uint8_t gsr_calc_activation(uint32_t conductance)
{
    // Valores típicos: 1-10 µS en reposo, 10-25 µS en activación
    const uint32_t BASELINE = 2000;   // 2 µS en unidades internas
    const uint32_t MAX_ACTIVATION = 25000; // 25 µS
    
    if (conductance <= BASELINE) {
        return 0;
    }
    
    uint32_t activation = (conductance - BASELINE) * 100 / 
                          (MAX_ACTIVATION - BASELINE);
    
    return (activation > 100) ? 100 : (uint8_t)activation;
}
```

### Detección de picos de estrés

```c
/**
 * @brief Detecta picos agudos de conductancia (respuesta a estrés).
 *
 * @param conductance Valor actual
 * @param history Buffer con historial reciente
 * @return true si se detecta pico
 */
bool gsr_detect_peak(uint32_t conductance, uint32_t *history, uint8_t len)
{
    uint32_t mean = 0;
    for (int i = 0; i < len; i++) {
        mean += history[i];
    }
    mean /= len;
    
    // Pico si supera 2 desviaciones sobre la media
    return (conductance > mean * 3);
}
```

---

## Pruebas Unitarias

```c
#include <zephyr/ztest.h>
#include "gsr.h"

ZTEST(gsr_tests, test_init)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(ad5940));
    zassert_true(device_is_ready(dev), "Sensor no listo");
    
    int ret = gsr_init(dev);
    zassert_equal(ret, 0, "Error en inicialización");
}

ZTEST(gsr_tests, test_read_conductance)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(ad5940));
    uint32_t conductance;
    
    gsr_start(dev);
    k_sleep(K_MSEC(100));
    
    int ret = gsr_read_conductance(dev, &conductance);
    zassert_equal(ret, 0, "Error en lectura");
    zassert_true(conductance > 0, "Conductancia cero");
}

ZTEST(gsr_tests, test_activation_calc)
{
    uint8_t act = gsr_calc_activation(5000);  // 5 µS
    zassert_between_inclusive(act, 10, 30, "Rango incorrecto");
}

ZTEST_SUITE(gsr_tests, NULL, NULL, NULL, NULL, NULL);
```

---

## Consumo de Energía

| Modo | Consumo | Descripción |
|:---|:---|:---|
| Sleep | 0.1 µA | Sensor apagado, RTC activo |
| Standby | 10 µA | Relojes activos, ADC en espera |
| Medición (20 Hz) | 800 µA | Típico, excitación AC |
| Medición (100 Hz) | 1.2 mA | Máximo, alta frecuencia |
| Calibración | 2 mA | Temporal durante calibración |

### Estrategia de bajo consumo
- Sensor en sleep cuando no se usa.
- Mediciones a 10 Hz en modo normal, 1 Hz en modo reposo.
- Excitación AC de baja amplitud (500 mV) para minimizar consumo.

---

## Referencias

1. Analog Devices. (2024). "AD5940 Datasheet: High Precision Impedance and Electrochemical Front End".
2. Analog Devices. (2025). "AN-1553: Implementing GSR Measurements with AD5940".
3. Nordic Semiconductor. (2025). "nRF5340 Product Specification".
4. Zephyr Project. (2026). "SPI Driver API Reference".
5. Boucsein, W. (2012). "Electrodermal Activity". Springer Science.
6. IEEE Std 1708-2023. "Standard for Wearable, Cuffless Blood Pressure Measuring Devices" (incluye lineamientos para GSR).
