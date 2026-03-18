# Driver Acelerómetro - LIS3DH

## Descripción

Este driver controla el acelerómetro **LIS3DH** de STMicroelectronics, un sensor MEMS de 3 ejes de ultra bajo consumo. En Memoria Fija v2 se utiliza para medir:

- **Actividad motora del paciente**
- **Agitación** (movimientos bruscos o repetitivos)
- **Postura** (detecta si el paciente está acostado, sentado o de pie)
- **Caídas** (detección de impactos)

El sensor se comunica vía I2C o SPI con el microcontrolador nRF5340. En esta implementación se utiliza **I2C** para simplificar el cableado.

---

## Especificaciones del Sensor

| Parámetro | Valor |
|:---|:---|
| Modelo | LIS3DH |
| Fabricante | STMicroelectronics |
| Interfaz | I2C (hasta 400 kHz) / SPI |
| Dirección I2C | 0x18 (SDO bajo) / 0x19 (SDO alto) |
| Ejes | 3 (X, Y, Z) |
| Rango | ±2g / ±4g / ±8g / ±16g |
| Resolución | 16 bits |
| Frecuencia muestreo | 1 Hz - 5 kHz |
| FIFO | 32 niveles |
| Interrupciones | 2 pines programables |
| Voltaje operación | 1.71V - 3.6V |
| Consumo | 2 µA (bajo consumo) / 11 µA (normal) / 165 µA (alto rendimiento) |

---

## Archivos del Driver

```
accelerometer/
├── accel.c          # Implementación del driver
├── accel.h          # Cabecera con API pública
└── README.md        # Este archivo
```

---

## API Pública

### Inicialización y configuración

```c
/**
 * @brief Inicializa el acelerómetro.
 *
 * @param dev Puntero al dispositivo (obtenido de Device Tree)
 * @return 0 en éxito, negativo en error
 */
int accel_init(const struct device *dev);

/**
 * @brief Configura los parámetros del sensor.
 *
 * @param dev Puntero al dispositivo
 * @param odr Frecuencia de salida (Hz)
 * @param range Rango (±2,4,8,16 g)
 * @param mode Modo de operación (low-power / normal / high-res)
 * @return 0 en éxito, negativo en error
 */
int accel_configure(const struct device *dev, uint16_t odr, uint8_t range, uint8_t mode);

/**
 * @brief Activa el sensor para comenzar la captura.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int accel_start(const struct device *dev);

/**
 * @brief Detiene la captura y pone el sensor en bajo consumo.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int accel_stop(const struct device *dev);
```

### Lectura de datos

```c
/**
 * @brief Lee una muestra de aceleración en los 3 ejes.
 *
 * @param dev Puntero al dispositivo
 * @param x Puntero para almacenar valor X (mg)
 * @param y Puntero para almacenar valor Y (mg)
 * @param z Puntero para almacenar valor Z (mg)
 * @return 0 en éxito, negativo en error
 */
int accel_read_sample(const struct device *dev, int16_t *x, int16_t *y, int16_t *z);

/**
 * @brief Lee múltiples muestras del FIFO.
 *
 * @param dev Puntero al dispositivo
 * @param buffer Buffer para almacenar muestras (formato X,Y,Z consecutivos)
 * @param samples Número de muestras a leer
 * @return Número de muestras leídas, negativo en error
 */
int accel_read_fifo(const struct device *dev, int16_t *buffer, uint16_t samples);

/**
 * @brief Lee la temperatura interna del sensor.
 *
 * @param dev Puntero al dispositivo
 * @param temp Puntero para almacenar temperatura (°C)
 * @return 0 en éxito, negativo en error
 */
int accel_read_temperature(const struct device *dev, float *temp);
```

### Procesamiento de datos

```c
/**
 * @brief Calcula la magnitud del vector de aceleración.
 *
 * @param x Aceleración en X (mg)
 * @param y Aceleración en Y (mg)
 * @param z Aceleración en Z (mg)
 * @return Magnitud (mg)
 */
uint16_t accel_magnitude(int16_t x, int16_t y, int16_t z);

/**
 * @brief Determina la postura del paciente.
 *
 * @param x Aceleración en X (mg)
 * @param y Aceleración en Y (mg)
 * @param z Aceleración en Z (mg)
 * @return Postura: 0=desconocida, 1=de pie, 2=sentado, 3=acostado
 */
uint8_t accel_posture(int16_t x, int16_t y, int16_t z);

/**
 * @brief Calcula el nivel de actividad.
 *
 * @param accel_buffer Buffer con muestras recientes
 * @param len Número de muestras
 * @return Nivel de actividad (0-100)
 */
uint8_t accel_activity_level(int16_t *accel_buffer, uint16_t len);

/**
 * @brief Detecta si ocurrió una caída.
 *
 * @param x Buffer de X
 * @param y Buffer de Y
 * @param z Buffer de Z
 * @param len Longitud del buffer
 * @return true si se detectó caída
 */
bool accel_fall_detected(int16_t *x, int16_t *y, int16_t *z, uint16_t len);
```

### Interrupciones

```c
/**
 * @brief Configura la detección de movimiento.
 *
 * @param dev Puntero al dispositivo
 * @param threshold Umbral de movimiento (mg)
 * @param duration Duración mínima (ms)
 * @return 0 en éxito, negativo en error
 */
int accel_set_motion_detection(const struct device *dev, uint16_t threshold, uint16_t duration);

/**
 * @brief Configura la detección de inactividad.
 *
 * @param dev Puntero al dispositivo
 * @param threshold Umbral de inactividad (mg)
 * @param duration Duración mínima (ms)
 * @return 0 en éxito, negativo en error
 */
int accel_set_inactivity_detection(const struct device *dev, uint16_t threshold, uint16_t duration);

/**
 * @brief Lee el estado de las interrupciones.
 *
 * @param dev Puntero al dispositivo
 * @return Máscara de interrupciones activas
 */
uint8_t accel_get_interrupt_status(const struct device *dev);
```

---

## Configuración Device Tree

En el archivo de Device Tree (`memoria_fija_v2.dts`):

```dts
&i2c2 {
    status = "okay";
    clock-frequency = <I2C_BITRATE_FAST>;
    
    lis3dh: lis3dh@18 {
        compatible = "st,lis3dh";
        reg = <0x18>;
        label = "ACCELEROMETER";
        irq-gpios = <&gpio0 22 GPIO_ACTIVE_HIGH>,
                    <&gpio0 23 GPIO_ACTIVE_HIGH>;
    };
};
```

---

## Ejemplo de Uso

```c
#include <zephyr/kernel.h>
#include "accel.h"

void accel_thread(void)
{
    const struct device *accel_dev;
    int16_t x, y, z;
    uint8_t posture;
    
    // Obtener dispositivo desde Device Tree
    accel_dev = DEVICE_DT_GET(DT_NODELABEL(lis3dh));
    if (!device_is_ready(accel_dev)) {
        printk("Error: acelerómetro no disponible\n");
        return;
    }
    
    // Inicializar y configurar
    accel_init(accel_dev);
    accel_configure(accel_dev, 50, 2, ACCEL_MODE_NORMAL);
    
    // Configurar detección de movimiento
    accel_set_motion_detection(accel_dev, 50, 1000);  // 50 mg durante 1s
    
    // Iniciar captura
    accel_start(accel_dev);
    
    while (1) {
        // Leer muestra
        accel_read_sample(accel_dev, &x, &y, &z);
        
        // Determinar postura
        posture = accel_posture(x, y, z);
        
        // Calcular actividad
        uint16_t mag = accel_magnitude(x, y, z);
        uint8_t activity = (mag > 1000) ? 100 : (mag * 100 / 1000);
        
        // Verificar interrupciones
        uint8_t int_status = accel_get_interrupt_status(accel_dev);
        if (int_status & ACCEL_INT_MOTION) {
            printk("Movimiento detectado!\n");
        }
        
        // Enviar a algoritmo de detección
        process_activity_data(activity, posture);
        
        k_sleep(K_MSEC(20));  // 50 Hz
    }
}
```

---

## Algoritmos de Procesamiento

### Cálculo de postura

```c
uint8_t accel_posture(int16_t x, int16_t y, int16_t z)
{
    const int16_t THRESHOLD = 800;  // mg
    
    if (abs(z) > THRESHOLD) {
        // Eje Z dominante
        return (z > 0) ? POSTURE_STANDING : POSTURE_SITTING;
    } else if (abs(x) > THRESHOLD) {
        // Eje X dominante (acostado de lado)
        return POSTURE_LYING;
    } else if (abs(y) > THRESHOLD) {
        // Eje Y dominante (acostado boca arriba/abajo)
        return POSTURE_LYING;
    }
    
    return POSTURE_UNKNOWN;
}
```

### Detección de agitación

```c
bool accel_detect_agitation(int16_t *x, int16_t *y, int16_t *z, uint16_t len)
{
    uint32_t variance_x = 0, variance_y = 0, variance_z = 0;
    int32_t mean_x = 0, mean_y = 0, mean_z = 0;
    
    // Calcular media
    for (int i = 0; i < len; i++) {
        mean_x += x[i];
        mean_y += y[i];
        mean_z += z[i];
    }
    mean_x /= len;
    mean_y /= len;
    mean_z /= len;
    
    // Calcular varianza
    for (int i = 0; i < len; i++) {
        variance_x += (x[i] - mean_x) * (x[i] - mean_x);
        variance_y += (y[i] - mean_y) * (y[i] - mean_y);
        variance_z += (z[i] - mean_z) * (z[i] - mean_z);
    }
    variance_x /= len;
    variance_y /= len;
    variance_z /= len;
    
    // Agitación si varianza alta en cualquier eje
    uint32_t total_variance = variance_x + variance_y + variance_z;
    return total_variance > AGITATION_THRESHOLD;
}
```

---

## Pruebas Unitarias

```c
#include <zephyr/ztest.h>
#include "accel.h"

ZTEST(accel_tests, test_init)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(lis3dh));
    zassert_true(device_is_ready(dev), "Sensor no listo");
    
    int ret = accel_init(dev);
    zassert_equal(ret, 0, "Error en inicialización");
}

ZTEST(accel_tests, test_read_sample)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(lis3dh));
    int16_t x, y, z;
    
    accel_start(dev);
    k_sleep(K_MSEC(100));
    
    int ret = accel_read_sample(dev, &x, &y, &z);
    zassert_equal(ret, 0, "Error en lectura");
    zassert_between_inclusive(x, -2000, 2000, "X fuera de rango");
}

ZTEST(accel_tests, test_posture)
{
    // De pie (gravedad en Z positivo)
    uint8_t p = accel_posture(0, 0, 1000);
    zassert_equal(p, POSTURE_STANDING, "Postura incorrecta");
    
    // Acostado (gravedad en X)
    p = accel_posture(1000, 0, 0);
    zassert_equal(p, POSTURE_LYING, "Postura incorrecta");
}

ZTEST_SUITE(accel_tests, NULL, NULL, NULL, NULL, NULL);
```

---

## Consumo de Energía

| Modo | Consumo | Descripción |
|:---|:---|:---|
| Power-down | 0.5 µA | Sensor apagado |
| Low-power (50 Hz) | 6 µA | Modo bajo consumo, 50 Hz |
| Low-power (1 Hz) | 2 µA | Monitoreo de inactividad |
| Normal (50 Hz) | 11 µA | Modo normal, 50 Hz |
| High-res (50 Hz) | 165 µA | Alta resolución, 50 Hz |

### Estrategia de bajo consumo
- Modo low-power para monitoreo continuo de actividad.
- Aumento a normal cuando se detecta movimiento.
- FIFO de 32 niveles permite procesar ráfagas sin despertar el MCU.

---

## Referencias

1. STMicroelectronics. (2024). "LIS3DH Datasheet: MEMS digital output motion sensor".
2. STMicroelectronics. (2025). "AN3308: LIS3DH application note".
3. Nordic Semiconductor. (2025). "nRF5340 Product Specification".
4. Zephyr Project. (2026). "I2C Driver API Reference".
5. IEEE Std 802.15.6-2023. "Wireless Body Area Networks" (incluye lineamientos para sensores de movimiento).
