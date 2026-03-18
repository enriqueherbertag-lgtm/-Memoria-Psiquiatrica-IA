# Conectividad Dual - Memoria Fija v2.1

## Descripción

El sistema de conectividad dual permite que el dispositivo conmute automáticamente entre Bluetooth LE (corto alcance, bajo consumo) y LTE-M (largo alcance, cobertura global) según la disponibilidad y el contexto del paciente.

---

## Estrategia de Conmutación

```
[PACIENTE EN HOSPITAL]
    ↓
[Bluetooth LE disponible] → Conexión directa a gateway hospitalario
    ↓
[Modo ahorro: transmisión cada 5 minutos]
    ↓
[ALTA HOSPITALARIA]
    ↓
[Bluetooth LE no disponible]
    ↓
[CONMUTACIÓN AUTOMÁTICA A LTE-M]
    ↓
[Modo ambulatorio: transmisión cada hora]
```

---

## Lógica de Conmutación

```c
/**
 * @brief Gestiona la conectividad dual del dispositivo.
 * 
 * Prioridades:
 * 1. Bluetooth LE (si hay gateway hospitalario cerca)
 * 2. WiFi conocido (redes guardadas del domicilio)
 * 3. LTE-M (cobertura global)
 */
void connectivity_manager_task(void)
{
    while (1) {
        // 1. Intentar Bluetooth (alcance 10m)
        if (bluetooth_scan_hospital_gateway()) {
            if (current_mode != MODE_BLUETOOTH) {
                switch_to_bluetooth();
                set_transmission_interval(TRANSMIT_5MIN);
            }
            k_sleep(K_SECONDS(60));
            continue;
        }
        
        // 2. Intentar WiFi conocido (redes guardadas)
        if (wifi_scan_known_networks()) {
            if (current_mode != MODE_WIFI) {
                switch_to_wifi();
                set_transmission_interval(TRANSMIT_15MIN);
            }
            k_sleep(K_SECONDS(60));
            continue;
        }
        
        // 3. Usar LTE-M (cobertura global)
        if (current_mode != MODE_CELLULAR) {
            switch_to_cellular();
            set_transmission_interval(TRANSMIT_1HOUR);
        }
        
        k_sleep(K_SECONDS(60));  // Revisar cada minuto
    }
}

/**
 * @brief Cambia a modo Bluetooth.
 */
void switch_to_bluetooth(void)
{
    // Apagar módulo celular
    cellular_power_off();
    
    // Activar Bluetooth
    bluetooth_power_on();
    bluetooth_advertising_start();
    
    current_mode = MODE_BLUETOOTH;
    log_connectivity_change("Bluetooth activado");
}

/**
 * @brief Cambia a modo LTE-M.
 */
void switch_to_cellular(void)
{
    // Apagar Bluetooth
    bluetooth_power_off();
    
    // Activar módulo celular
    cellular_power_on();
    cellular_connect(HEALTH_APN);
    
    current_mode = MODE_CELLULAR;
    log_connectivity_change("LTE-M activado");
}
```

---

## Consumo Energético por Modo

| Modo | Transmisión | Consumo promedio | Autonomía (batería 200 mAh) |
|:---|:---|:---|:---|
| Bluetooth | Cada 5 min | 15 µA | 15 días |
| WiFi | Cada 15 min | 25 µA | 9 días |
| LTE-M | Cada 1 hora | 35 µA | 7 días |

---

## Indicadores para el Paciente

| LED | Significado |
|:---|:---|
| Verde fijo | Bluetooth conectado (hospital) |
| Verde intermitente | Buscando Bluetooth/WiFi |
| Azul fijo | LTE-M conectado (modo ambulatorio) |
| Azul intermitente | Buscando red celular |
| Rojo | Error de conectividad |

---

## Historial de Conectividad

```c
typedef struct {
    uint32_t timestamp;
    uint8_t mode;           // 0: Bluetooth, 1: WiFi, 2: LTE-M
    int8_t rssi;            // Intensidad de señal (dBm)
    uint8_t battery_level;  // Nivel al momento del cambio
} ConnectivityLog_t;

/**
 * @brief Registra cambios de conectividad para diagnóstico.
 */
void log_connectivity_change(char *reason)
{
    ConnectivityLog_t log = {
        .timestamp = get_epoch_time(),
        .mode = current_mode,
        .rssi = get_current_rssi(),
        .battery_level = battery_get_percent()
    };
    
    storage_append(CONNECTIVITY_LOG_ADDR, &log, sizeof(log));
}
```
