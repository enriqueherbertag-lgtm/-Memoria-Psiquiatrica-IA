# Driver Batería - MEC225 + MAX77756

## Descripción

Este driver gestiona el sistema de alimentación de Memoria Fija v2, que combina:

- **Batería MEC225** de EaglePicher (Li-ion biocompatible de 200 mAh)
- **PMIC MAX77756** de Maxim Integrated (gestión de carga y alimentación)
- **Fuel gauge** integrado en el PMIC para medición precisa de estado de carga

El sistema proporciona:
- Medición precisa del estado de carga (SoC, State of Charge)
- Monitorización de voltaje, corriente y temperatura
- Gestión de carga inalámbrica (Qi)
- Protecciones contra sobrecarga, sobredescarga y sobrecorriente
- Estimación de autonomía restante

La comunicación con el PMIC se realiza vía I2C.

---

## Especificaciones del Sistema

### Batería (MEC225)
| Parámetro | Valor |
|:---|:---|
| Modelo | MEC225 |
| Fabricante | EaglePicher |
| Tipo | Li-ion (LiCoO₂) biocompatible |
| Capacidad | 200 mAh |
| Voltaje nominal | 3.7 V |
| Voltaje máx carga | 4.2 V |
| Voltaje mín descarga | 3.0 V |
| Ciclos vida | > 1000 (80% capacidad residual) |
| Dimensiones | 20 mm × 15 mm × 4 mm |
| Peso | 2.5 g |

### PMIC (MAX77756)
| Parámetro | Valor |
|:---|:---|
| Modelo | MAX77756 |
| Fabricante | Maxim Integrated |
| Interfaz | I2C (hasta 400 kHz) |
| Dirección I2C | 0x50 (por defecto) |
| Eficiencia carga | 92% |
| Corriente carga | 50 mA (máx) |
| Protecciones | Sobrevoltaje, sobrecorriente, temperatura |
| Fuel gauge | Coulomb counter integrado |
| Consumo propio | 25 µA (operación) / 5 µA (sleep) |

### Carga inalámbrica (Qi)
| Parámetro | Valor |
|:---|:---|
| Estándar | Qi (baja potencia) |
| Frecuencia | 100-200 kHz |
| Potencia recepción | 250 mW (máx) |
| Bobina | 10 mm × 10 mm × 1 mm (ferrita) |
| Distancia carga | 10 mm (máx) |
| Eficiencia | 70-80% |

---

## Archivos del Driver

```
battery/
├── battery.c          # Implementación del driver
├── battery.h          # Cabecera con API pública
└── README.md          # Este archivo
```

---

## API Pública

### Inicialización y configuración

```c
/**
 * @brief Inicializa el sistema de batería.
 *
 * @param dev Puntero al dispositivo (obtenido de Device Tree)
 * @return 0 en éxito, negativo en error
 */
int battery_init(const struct device *dev);

/**
 * @brief Configura los parámetros de la batería.
 *
 * @param dev Puntero al dispositivo
 * @param capacity Capacidad nominal (mAh)
 * @param vmax Voltaje máx carga (mV)
 * @param vmin Voltaje mín descarga (mV)
 * @return 0 en éxito, negativo en error
 */
int battery_configure(const struct device *dev, uint16_t capacity, uint16_t vmax, uint16_t vmin);

/**
 * @brief Activa el monitoreo continuo.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int battery_start_monitoring(const struct device *dev);

/**
 * @brief Detiene el monitoreo (bajo consumo).
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int battery_stop_monitoring(const struct device *dev);
```

### Estado de la batería

```c
/**
 * @brief Obtiene el voltaje actual de la batería.
 *
 * @param dev Puntero al dispositivo
 * @param voltage_mv Puntero para almacenar voltaje (mV)
 * @return 0 en éxito, negativo en error
 */
int battery_get_voltage(const struct device *dev, uint16_t *voltage_mv);

/**
 * @brief Obtiene la corriente actual.
 *
 * @param dev Puntero al dispositivo
 * @param current_ma Puntero para almacenar corriente (mA, positivo = carga, negativo = descarga)
 * @return 0 en éxito, negativo en error
 */
int battery_get_current(const struct device *dev, int16_t *current_ma);

/**
 * @brief Obtiene la temperatura de la batería.
 *
 * @param dev Puntero al dispositivo
 * @param temp_c Puntero para almacenar temperatura (°C)
 * @return 0 en éxito, negativo en error
 */
int battery_get_temperature(const struct device *dev, float *temp_c);

/**
 * @brief Obtiene el estado de carga (SoC).
 *
 * @param dev Puntero al dispositivo
 * @param soc_pct Puntero para almacenar porcentaje (0-100)
 * @return 0 en éxito, negativo en error
 */
int battery_get_soc(const struct device *dev, uint8_t *soc_pct);

/**
 * @brief Obtiene el estado de salud (SoH) de la batería.
 *
 * @param dev Puntero al dispositivo
 * @param soh_pct Puntero para almacenar porcentaje (0-100)
 * @return 0 en éxito, negativo en error
 */
int battery_get_soh(const struct device *dev, uint8_t *soh_pct);

/**
 * @brief Obtiene el tiempo restante estimado.
 *
 * @param dev Puntero al dispositivo
 * @param hours Puntero para almacenar horas restantes
 * @return 0 en éxito, negativo en error
 */
int battery_get_time_remaining(const struct device *dev, float *hours);
```

### Carga

```c
/**
 * @brief Verifica si la batería está en carga.
 *
 * @param dev Puntero al dispositivo
 * @param charging Puntero para almacenar estado (true = cargando)
 * @return 0 en éxito, negativo en error
 */
int battery_is_charging(const struct device *dev, bool *charging);

/**
 * @brief Verifica si la carga está completa.
 *
 * @param dev Puntero al dispositivo
 * @param full Puntero para almacenar estado (true = completa)
 * @return 0 en éxito, negativo en error
 */
int battery_is_full(const struct device *dev, bool *full);

/**
 * @brief Obtiene el tiempo estimado hasta carga completa.
 *
 * @param dev Puntero al dispositivo
 * @param minutes Puntero para almacenar minutos restantes
 * @return 0 en éxito, negativo en error
 */
int battery_get_charge_time(const struct device *dev, uint16_t *minutes);

/**
 * @brief Inicia la carga (si está conectado cargador).
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int battery_start_charging(const struct device *dev);

/**
 * @brief Detiene la carga manualmente.
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int battery_stop_charging(const struct device *dev);
```

### Alarmas y protección

```c
/**
 * @brief Configura umbrales de alarma.
 *
 * @param dev Puntero al dispositivo
 * @param low_soc Umbral de batería baja (%) 
 * @param critical_soc Umbral crítico (%)
 * @param high_temp Umbral de temperatura alta (°C)
 * @return 0 en éxito, negativo en error
 */
int battery_set_alarms(const struct device *dev, uint8_t low_soc, uint8_t critical_soc, float high_temp);

/**
 * @brief Obtiene el estado de las alarmas.
 *
 * @param dev Puntero al dispositivo
 * @return Máscara de alarmas activas (bit0 = batería baja, bit1 = crítica, bit2 = temperatura alta)
 */
uint8_t battery_get_alarms(const struct device *dev);

/**
 * @brief Verifica si la batería está en estado crítico (apagado inminente).
 *
 * @param dev Puntero al dispositivo
 * @param critical Puntero para almacenar estado
 * @return 0 en éxito, negativo en error
 */
int battery_is_critical(const struct device *dev, bool *critical);
```

### Calibración y mantenimiento

```c
/**
 * @brief Calibra el fuel gauge (aprendizaje de capacidad real).
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int battery_calibrate(const struct device *dev);

/**
 * @brief Reinicia el contador de ciclos (después de reemplazo de batería).
 *
 * @param dev Puntero al dispositivo
 * @return 0 en éxito, negativo en error
 */
int battery_reset_cycles(const struct device *dev);

/**
 * @brief Obtiene el número de ciclos de carga completados.
 *
 * @param dev Puntero al dispositivo
 * @param cycles Puntero para almacenar número de ciclos
 * @return 0 en éxito, negativo en error
 */
int battery_get_cycles(const struct device *dev, uint16_t *cycles);
```

---

## Configuración Device Tree

En el archivo de Device Tree (`memoria_fija_v2.dts`):

```dts
&i2c1 {
    status = "okay";
    clock-frequency = <I2C_BITRATE_FAST>;
    
    max77756: max77756@50 {
        compatible = "maxim,max77756";
        reg = <0x50>;
        label = "BATTERY_PMIC";
        battery-capacity = <200>;      /* mAh */
        charge-current = <50>;          /* mA */
        vmax = <4200>;                  /* mV */
        vmin = <3000>;                  /* mV */
        low-battery = <15>;             /* % */
        critical-battery = <5>;          /* % */
        temp-max = <45>;                 /* °C */
    };
};
```

---

## Ejemplo de Uso

```c
#include <zephyr/kernel.h>
#include "battery.h"

void battery_monitor_thread(void)
{
    const struct device *bat_dev;
    uint8_t soc;
    uint16_t voltage;
    bool critical;
    
    // Obtener dispositivo desde Device Tree
    bat_dev = DEVICE_DT_GET(DT_NODELABEL(max77756));
    if (!device_is_ready(bat_dev)) {
        printk("Error: sistema de batería no disponible\n");
        return;
    }
    
    // Inicializar y configurar
    battery_init(bat_dev);
    battery_configure(bat_dev, 200, 4200, 3000);
    battery_set_alarms(bat_dev, 15, 5, 45);
    battery_start_monitoring(bat_dev);
    
    while (1) {
        // Leer estado cada minuto
        battery_get_soc(bat_dev, &soc);
        battery_get_voltage(bat_dev, &voltage);
        battery_is_critical(bat_dev, &critical);
        
        printk("Batería: %d%%, %d mV\n", soc, voltage);
        
        // Verificar alarmas
        uint8_t alarms = battery_get_alarms(bat_dev);
        if (alarms & BATTERY_ALARM_LOW) {
            printk("ALERTA: Batería baja (%d%%)\n", soc);
            notify_user(BATTERY_LOW);
        }
        
        if (alarms & BATTERY_ALARM_CRITICAL) {
            printk("ALERTA CRÍTICA: Batería agotada\n");
            // Preparar sistema para apagado ordenado
            system_shutdown();
        }
        
        if (critical) {
            printk("ESTADO CRÍTICO: Apagado inminente\n");
        }
        
        k_sleep(K_SECONDS(60));
    }
}

// Interrupción de carga inalámbrica
void wireless_charge_callback(const struct device *dev)
{
    bool charging;
    battery_is_charging(dev, &charging);
    
    if (charging) {
        printk("Carga inalámbrica iniciada\n");
        // Activar indicador LED
        gpio_pin_set(led_dev, LED_CHARGE_PIN, 1);
    } else {
        printk("Carga inalámbrica finalizada\n");
        gpio_pin_set(led_dev, LED_CHARGE_PIN, 0);
    }
}
```

---

## Algoritmos de Estimación

### Cálculo de SoC (Coulomb counting + corrección por voltaje)

```c
uint8_t battery_calculate_soc(int32_t coulomb_count, uint16_t voltage_mv, uint16_t capacity_mah)
{
    // SoC base por coulomb counting
    int32_t coulomb_soc = (coulomb_count * 100) / (capacity_mah * 3600);
    
    // Corrección por voltaje en extremos
    if (coulomb_soc > 90 || coulomb_soc < 10) {
        uint8_t voltage_soc = 0;
        
        if (voltage_mv >= 4100) {
            voltage_soc = 100;
        } else if (voltage_mv <= 3300) {
            voltage_soc = 0;
        } else {
            // Interpolación lineal en zona media
            voltage_soc = (voltage_mv - 3300) * 100 / (4100 - 3300);
        }
        
        // Promedio ponderado: 30% coulomb, 70% voltaje en extremos
        return (coulomb_soc * 30 + voltage_soc * 70) / 100;
    }
    
    return coulomb_soc;
}
```

### Estimación de tiempo restante

```c
float battery_time_remaining(uint8_t soc, int16_t current_ma)
{
    if (current_ma >= 0) {
        // En carga - no aplica tiempo restante
        return -1.0;
    }
    
    if (current_ma == 0) {
        return 999.0;  // Tiempo indefinido
    }
    
    // corriente negativa = descarga
    float discharge_current = -current_ma / 1000.0;  // A
    float capacity_remaining = (soc * 0.2) / 100.0;  // Ah (200 mAh = 0.2 Ah)
    
    return capacity_remaining / discharge_current;  // horas
}
```

---

## Pruebas Unitarias

```c
#include <zephyr/ztest.h>
#include "battery.h"

ZTEST(battery_tests, test_init)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(max77756));
    zassert_true(device_is_ready(dev), "PMIC no listo");
    
    int ret = battery_init(dev);
    zassert_equal(ret, 0, "Error en inicialización");
}

ZTEST(battery_tests, test_read_voltage)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(max77756));
    uint16_t voltage;
    
    int ret = battery_get_voltage(dev, &voltage);
    zassert_equal(ret, 0, "Error leyendo voltaje");
    zassert_between_inclusive(voltage, 3000, 4200, "Voltaje fuera de rango");
}

ZTEST(battery_tests, test_soc)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(max77756));
    uint8_t soc;
    
    int ret = battery_get_soc(dev, &soc);
    zassert_equal(ret, 0, "Error leyendo SoC");
    zassert_between_inclusive(soc, 0, 100, "SoC fuera de rango");
}

ZTEST(battery_tests, test_alarms)
{
    const struct device *dev = DEVICE_DT_GET(DT_NODELABEL(max77756));
    
    battery_set_alarms(dev, 20, 5, 40);
    uint8_t alarms = battery_get_alarms(dev);
    
    // No deberían activarse en condiciones normales
    zassert_equal(alarms, 0, "Alarmas activadas incorrectamente");
}

ZTEST_SUITE(battery_tests, NULL, NULL, NULL, NULL, NULL);
```

---

## Consumo de Energía

| Modo | Consumo | Descripción |
|:---|:---|:---|
| Sleep (PMIC) | 5 µA | PMIC en modo ultra bajo consumo |
| Monitoreo básico | 25 µA | Lectura periódica de voltaje y temperatura |
| Monitoreo completo | 50 µA | Con fuel gauge activo |
| Carga activa | 50 mA + 5 mA (PMIC) | Corriente de carga + consumo PMIC |

### Estrategia de bajo consumo
- PMIC en sleep cuando el dispositivo está en modo avión.
- Lectura de SoC cada 60 segundos en modo normal.
- Fuel gauge activado solo cuando hay cambios significativos de corriente.

---

## Referencias

1. Maxim Integrated. (2024). "MAX77756 Datasheet: Low-Power Charger with Integrated Fuel Gauge".
2. EaglePicher. (2025). "MEC225 Medical Grade Lithium-Ion Cell Datasheet".
3. Wireless Power Consortium. (2024). "Qi Specification v1.3".
4. Nordic Semiconductor. (2025). "nRF5340 Product Specification".
5. Zephyr Project. (2026). "I2C Driver API Reference".
6. IEEE 1725-2023. "Standard for Rechargeable Batteries for Mobile Computing Devices".
7. IEC 62133-2:2023. "Secondary cells and batteries containing alkaline or other non-acid electrolytes".
