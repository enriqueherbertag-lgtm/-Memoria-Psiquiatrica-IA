# Comunicación Celular LTE-M/NB-IoT - Memoria Fija v2.1

## Descripción

Módulo de conectividad celular para pacientes ambulatorios, permitiendo comunicación continua fuera del entorno hospitalario sin depender de Bluetooth o WiFi.

---

## Especificaciones del Módulo

| Parámetro | Valor |
|:---|:---|
| Modelo | Quectel BG95-M3 |
| Tecnologías | LTE-M / NB-IoT / GSM |
| Bandas | LTE: B1/B2/B3/B4/B5/B8/B12/B13/B18/B19/B20/B25/B26/B28/B66/B85 |
| Consumo sleep | 1.5 µA |
| Consumo transmisión | 200 mA (pico) |
| Voltaje | 2.1V - 3.6V |
| Interfaz | UART + AT commands |
| Seguridad | Cifrado extremo a extremo + APN privado |
| Certificaciones | GCF, FCC, CE, RoHS |

---

## Integración con nRF5340

```c
/**
 * @brief Inicializa módulo celular.
 *
 * @param uart_dev Dispositivo UART para comunicación
 * @return 0 en éxito, negativo en error
 */
int cellular_init(const struct device *uart_dev)
{
    // Configurar UART
    struct uart_config uart_cfg = {
        .baudrate = 115200,
        .parity = UART_CFG_PARITY_NONE,
        .stop_bits = UART_CFG_STOP_BITS_1,
        .data_bits = UART_CFG_DATA_BITS_8,
        .flow_ctrl = UART_CFG_FLOW_CTRL_NONE
    };
    uart_configure(uart_dev, &uart_cfg);
    
    // Enviar AT para verificar comunicación
    char *cmd = "AT\r\n";
    uart_tx(uart_dev, cmd, strlen(cmd), SYS_FOREVER_MS);
    
    // Esperar respuesta "OK"
    char response[32];
    uart_rx(uart_dev, response, sizeof(response), SYS_FOREVER_MS);
    
    if (strstr(response, "OK") == NULL) {
        return -ENODEV;
    }
    
    return 0;
}

/**
 * @brief Conecta a red celular.
 *
 * @param apn APN privado de salud
 * @return 0 en éxito, negativo en error
 */
int cellular_connect(const char *apn)
{
    // Configurar APN
    char cmd[64];
    sprintf(cmd, "AT+CGDCONT=1,\"IP\",\"%s\"\r\n", apn);
    cellular_send_command(cmd);
    
    // Activar contexto PDP
    cellular_send_command("AT+CGACT=1,1\r\n");
    
    // Verificar conexión
    char response[64];
    cellular_send_command("AT+CGREG?\r\n");
    
    return cellular_check_registration(response);
}

/**
 * @brief Envía datos vía MQTT-SN.
 *
 * @param data Datos a enviar
 * @param len Longitud de datos
 * @return 0 en éxito, negativo en error
 */
int cellular_send_data(uint8_t *data, uint16_t len)
{
    // Implementación MQTT-SN sobre UDP
    return mqtt_sn_publish(data, len);
}
```

---

## Selección Automática de Red

```c
/**
 * @brief Selecciona automáticamente la mejor red disponible.
 *
 * Prioridad: 1. Bluetooth (hospital), 2. WiFi (domicilio), 3. LTE-M
 */
void connectivity_manager(void)
{
    while (1) {
        // 1. Intentar Bluetooth (alcance 10m)
        if (bluetooth_scan_hospital_gateway()) {
            switch_to_bluetooth();
            k_sleep(K_SECONDS(300));  // Reintentar cada 5 min
            continue;
        }
        
        // 2. Intentar WiFi conocido (redes guardadas)
        if (wifi_scan_known_networks()) {
            switch_to_wifi();
            k_sleep(K_SECONDS(300));
            continue;
        }
        
        // 3. Usar LTE-M
        if (!cellular_connected) {
            cellular_connect(HEALTH_APN);
            switch_to_cellular();
        }
        
        k_sleep(K_SECONDS(60));  // Revisar cada minuto
    }
}
```

---

## Consumo Energético Optimizado

| Modo | Consumo | Descripción |
|:---|:---|:---|
| Sleep profundo | 2 µA | Módulo apagado, solo RTC |
| PSM (Power Saving Mode) | 5 µA | Registrado en red, pero inactivo |
| eDRX extendido | 15 µA | Escucha cada 10 minutos |
| Transmisión | 200 mA | Envío de datos (100 ms) |

**Estrategia:** 1 transmisión/hora → consumo promedio **< 50 µA**
