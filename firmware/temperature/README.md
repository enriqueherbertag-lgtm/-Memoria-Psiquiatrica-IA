# Driver Temperatura - TMP117

## Descripción

Este driver controla el sensor de temperatura **TMP117** de Texas Instruments, un sensor de alta precisión diseñado específicamente para aplicaciones médicas. En Memoria Fija v2 se utiliza para:

- **Monitorización de temperatura corporal**
- **Compensación de otros sensores** (GSR, PPG) por variaciones térmicas
- **Detección de fiebre** como posible indicador de infección o estrés fisiológico
- **Supervisión de seguridad** del dispositivo (sobrecalentamiento)

El sensor se comunica vía I2C con el microcontrolador nRF5340.

---

## Especificaciones del Sensor

| Parámetro | Valor |
|:---|:---|
| Modelo | TMP117 |
| Fabricante | Texas Instruments |
| Interfaz | I2C (hasta 1 MHz) |
| Dirección I2C | 0x48 (por defecto, configurable hasta 0x4F) |
| Precisión | ±0.1°C (de 30°C a 45°C) |
| Resolución | 16 bits (0.0078125°C) |
| Rango | -20°C a +50°C |
| Tiempo de conversión | 15.5 ms (modo estándar) |
| Voltaje operación | 1.8V - 5.5V |
| Consumo | 3.5 µA (una conversión/segundo) / 0.1 µA (shutdown) |
| Calibración | De fábrica (trazable NIST) |

---

## Archivos del Driver

```
temperature/
├── temp.c          # Implementación del driver
├── temp.h          # Cabecera con API pública
└── README.md       # Este archivo
```

---

## API Pública

### Inicialización y configuración

```c
/**
 * @brief Inicializa el sensor de temperatura.
 *
 * @param dev Puntero al dispositivo (obtenido de Device Tree)
 * @return 0 en éxito, negativo en error
 */
int temp_init(const struct device *dev);

/**
 * @brief Configura los parámetros del sensor.
 *
 * @param dev Puntero al dispositivo
 * @param conversion_rate Frecuencia de conversión (Hz)
 * @param averaging Modo de promediado (1, 8, 32, 64 muestras)
 * @return 0 en éxito, negativo en error
 */
int temp_configure(const struct device *dev, uint8_t conversion_rate, uint8_t averaging);

/**
 * @brief Activa el sensor para comenzar la captura.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int temp_start(const struct device *dev);

/**
 * @brief Detiene la captura y pone el sensor en bajo consumo.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int temp_stop(const struct device *dev);
```

### Lectura de datos

```c
/**
 * @brief Lee una muestra de temperatura.
 *
 * @param dev Puntero al dispositivo
 * @param temp Puntero para almacenar temperatura (°C)
 * @return 0 en éxito, negativo en error
 */
int temp_read(const struct device *dev, float *temp);

/**
 * @brief Lee la temperatura en formato crudo (valor del registro).
 *
 * @param dev Puntero al dispositivo
 * @param raw Puntero para almacenar valor crudo (16 bits)
 * @return 0 en éxito, negativo en error
 */
int temp_read_raw(const struct device *dev, uint16_t *raw);

/**
 * @brief Lee múltiples muestras promediadas.
 *
 * @param dev Puntero al dispositivo
 * @param samples Número de muestras a promediar
 * @return Temperatura promediada (°C)
 */
float temp_read_averaged(const struct device *dev, uint8_t samples);
```

### Límites y alarmas

```c
/**
 * @brief Configura límites de temperatura para generar interrupción.
 *
 * @param dev Puntero al dispositivo
 * @param low_limit Límite inferior (°C)
 * @param high_limit Límite superior (°C)
 * @return 0 en éxito, negativo en error
 */
int temp_set_limits(const struct device *dev, float low_limit, float high_limit);

/**
 * @brief Lee el estado de las alarmas de temperatura.
 *
 * @param dev Puntero al dispositivo
 * @return Máscara: bit0 = bajo, bit1 = alto
 */
uint8_t temp_get_alert_status(const struct device *dev);

/**
 * @brief Detecta si hay fiebre (temperatura > 38°C).
 *
 * @param dev Puntero al dispositivo
 * @return true si hay fiebre
 */
bool temp_detect_fever(const struct device *dev);
```

### Compensación para otros sensores

```c
/**
 * @brief Obtiene factor de compensación para sensor GSR.
 *
 * La conductancia dérmica varía con la temperatura.
 *
 * @param temp Temperatura actual (°C)
 * @return Factor de compensación (multiplicador)
 */
float temp_gsr_compensation(float temp);

/**
 * @brief Obtiene factor de compensación para sensor PPG.
 *
 * El flujo sanguíneo y la absorción óptica varían con temperatura.
 *
 * @param temp Temperatura actual (°C)
 * @return Factor de compensación (multiplicador)
 */
float temp_ppg_compensation(float temp);
```

---

## Configuración Device Tree

En el archivo de Device Tree (`memoria_fija_v2.dts`):

```dts
&i2c1 {
    status = "okay";
    clock-frequency = <I2C_BITRATE_FAST>;
    
    tmp117: tmp117@48 {
        compatible = "ti,tmp117";
        reg = <0x48>;
        label = "TEMP_SENSOR";
        alert-gpios = <&gpio0 24 GPIO_ACTIVE_HIGH>;
    };
};
```

---

## Ejemplo de Uso

```c
#include <zephyr/kernel.h>
#include "temp.h"

void temperature_thread(void)
{
    const struct device *temp_dev;
    float temperature;
    
    // Obtener dispositivo desde Device Tree
    temp_dev = DEVICE_DT_GET(DT_NODELABEL(tmp117));
    if (!device_is_ready(temp_dev)) {
        printk("Error: sensor de temperatura no disponible\n");
        return;
    }
    
    // Inicializar y configurar
    temp_init(temp_dev);
    temp_configure(temp_dev, 1, 8);  // 1 Hz, promediado 8 muestras
    
    // Configurar límites de alarma
    temp_set_limits(temp_dev, 35.0, 38.5);  // Hipotermia < 35°C, fiebre > 38.5°C
    
    // Iniciar captura
    temp_start(temp_dev);
    
    while (1) {
        // Leer temperatura
        temp_read(temp_dev, &temperature);
        
        // Verificar alarmas
        uint8_t alert = temp_get_alert_status(temp_dev);
        if (alert & TEMP_ALERT_HIGH) {
            printk("ALERTA: Temperatura alta (fiebre?)\n");
            notify_medical_staff(TEMP_HIGH_ALERT);
        }
        
        if (alert & TEMP_ALERT_LOW) {
            printk("ALERTA: Temperatura baja (hipotermia?)\n");
            notify_medical_staff(TEMP_LOW_ALERT);
        }
        
        // Detectar fiebre
        if (temp_detect_fever(temp_dev)) {
            printk("Fiebre detectada: %.2f°C\n", temperature);
        }
        
        // Enviar a otros sistemas para compensación
        compensate_gsr(temperature);
        compensate_ppg(temperature);
        
        k_sleep(K_SECONDS(1));  // 1 Hz
    }
}
```

---

## Algoritmos de Compensación

### Compensación para GSR

```c
float temp_gsr_compensation(float temp)
{
    // Conductancia dérmica aumenta ~2% por °C sobre 37°C
    const float REF_TEMP = 37.0;
    const float COEFF = 0.02;
    
    if (temp <= REF_TEMP) {
        return 1.0;
    }
    
    float delta = temp - REF_TEMP;
    return 1.0 + (delta * COEFF);
}
```

### Compensación para PPG

```c
float temp_ppg_compensation(float temp)
{
    // Flujo sanguíneo varía con temperatura
    // Modelo simplificado basado en datos empíricos
    const float REF_TEMP = 37.0;
    
    if (temp < 35.0) {
        return 0.85;  // Vasoconstricción
    } else if (temp > 39.0) {
        return 1.15;  // Vasodilatación
    } else {
        // Interpolación lineal
        return 1.0 + (temp - REF_TEMP) * 0.05;
    }
}
```

---

## Pruebas Unitarias

```c
#include <zephyr/ztest.h>
#include "temp.h"

ZTEST(temp_tests, test_init)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(tmp117));
    zassert_true(device_is_ready(dev), "Sensor no listo");
    
    int ret = temp_init(dev);
    zassert_equal(ret, 0, "Error en inicialización");
}

ZTEST(temp_tests, test_read)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(tmp117));
    float temp;
    
    temp_start(dev);
    k_sleep(K_MSEC(100));
    
    int ret = temp_read(dev, &temp);
    zassert_equal(ret, 0, "Error en lectura");
    zassert_between_inclusive(temp, 30.0, 40.0, "Temperatura fuera de rango fisiológico");
}

ZTEST(temp_tests, test_limits)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(tmp117));
    
    temp_set_limits(dev, 36.0, 38.0);
    
    // Simular temperatura baja
    uint8_t alert = temp_get_alert_status(dev);
    zassert_true((alert & TEMP_ALERT_LOW) || !(alert & TEMP_ALERT_HIGH), "Límites incorrectos");
}

ZTEST(temp_tests, test_compensation)
{
    float factor = temp_gsr_compensation(38.0);
    zassert_between_inclusive(factor, 1.01, 1.03, "Factor de compensación incorrecto");
    
    factor = temp_ppg_compensation(38.0);
    zassert_between_inclusive(factor, 1.04, 1.06, "Factor de compensación incorrecto");
}

ZTEST_SUITE(temp_tests, NULL, NULL, NULL, NULL, NULL);
```

---

## Consumo de Energía

| Modo | Consumo | Descripción |
|:---|:---|:---|
| Shutdown | 0.1 µA | Sensor apagado |
| Una conversión/segundo | 3.5 µA | Modo normal, 1 Hz |
| Cuatro conversiones/segundo | 6.0 µA | 4 Hz |
| Conversión continua (125 ms) | 45 µA | Máxima frecuencia |
| Promediado (64 muestras) | +10% | Mayor consumo por cálculo |

### Estrategia de bajo consumo
- Modo shutdown cuando no se necesita temperatura.
- Una conversión por segundo suficiente para monitoreo fisiológico.
- Promediado solo cuando se requiere alta precisión.

---

## Referencias

1. Texas Instruments. (2024). "TMP117 Datasheet: High-Accuracy, Low-Power, Digital Temperature Sensor".
2. Texas Instruments. (2025). "SBOU210: TMP117 User's Guide".
3. Nordic Semiconductor. (2025). "nRF5340 Product Specification".
4. Zephyr Project. (2026). "I2C Driver API Reference".
5. ASTM E1112-23. "Standard Specification for Electronic Thermometer for Intermittent Determination of Patient Temperature".
6. ISO 80601-2-56:2023. "Medical electrical equipment - Particular requirements for basic safety and essential performance of clinical thermometers".
