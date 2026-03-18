# Firmware de Memoria Fija v2

Este directorio contiene el código fuente del firmware del dispositivo implantable Memoria Fija v2.

---

## Estructura del Directorio

```
firmware/
├── README.md
├── main/
│   ├── main.c
│   ├── main.h
│   └── CMakeLists.txt
├── drivers/
│   ├── ppg/
│   │   ├── ppg.c
│   │   ├── ppg.h
│   │   └── README.md
│   ├── gsr/
│   │   ├── gsr.c
│   │   ├── gsr.h
│   │   └── README.md
│   ├── accelerometer/
│   │   ├── accel.c
│   │   ├── accel.h
│   │   └── README.md
│   ├── temperature/
│   │   ├── temp.c
│   │   ├── temp.h
│   │   └── README.md
│   ├── piezo/
│   │   ├── piezo.c
│   │   ├── piezo.h
│   │   └── README.md
│   └── battery/
│       ├── battery.c
│       ├── battery.h
│       └── README.md
├── algorithm/
│   ├── detector.c
│   ├── detector.h
│   ├── adaptive.c
│   ├── adaptive.h
│   └── README.md
├── communication/
│   ├── ble.c
│   ├── ble.h
│   ├── security.c
│   ├── security.h
│   └── README.md
├── power/
│   ├── power.c
│   ├── power.h
│   └── README.md
├── storage/
│   ├── storage.c
│   ├── storage.h
│   └── README.md
├── bootloader/
│   ├── bootloader.c
│   ├── bootloader.h
│   └── README.md
├── tests/
│   ├── unit/
│   ├── integration/
│   └── hardware-in-loop/
├── tools/
│   ├── flash.py
│   ├── debug.py
│   └── log_parser.py
└── docs/
    ├── api-reference.md
    └── build-instructions.md
```

---

## Plataforma de Desarrollo

### Hardware objetivo
| Componente | Modelo |
|:---|:---|
| MCU principal | nRF5340 (Nordic Semiconductor) |
| Núcleo de aplicación | ARM Cortex-M33 (64 MHz) |
| Núcleo de red | ARM Cortex-M4F (64 MHz) |
| Memoria | 1 MB Flash + 512 KB RAM |

### Entorno de desarrollo
| Herramienta | Versión | Uso |
|:---|:---|:---|
| nRF Connect SDK | v2.8.0 | SDK principal |
| Zephyr RTOS | v3.7.0 | Sistema operativo en tiempo real |
| GCC ARM Embedded | 12.2.1 | Compilador |
| CMake | 3.26+ | Sistema de construcción |
| Ninja | 1.11+ | Ejecutor de builds |
| VS Code | 1.95+ | IDE recomendado |
| nRF Command Line Tools | 10.24.0 | Herramientas Nordic |

---

## Arquitectura del Firmware

### Capas del sistema
```
┌─────────────────────────────────────┐
│         Aplicación Principal         │
│  (main.c, detección, comunicación)  │
├─────────────────────────────────────┤
│         Algoritmos Especializados    │
│   (detector.c, adaptive.c)          │
├─────────────────────────────────────┤
│           Gestores de Dispositivos   │
│  (drivers/ - sensores, actuadores)  │
├─────────────────────────────────────┤
│          Sistema Operativo (Zephyr)  │
├─────────────────────────────────────┤
│                HAL (nRF)             │
├─────────────────────────────────────┤
│               Hardware               │
└─────────────────────────────────────┘
```

### Modelo de concurrencia
| Hilo | Prioridad | Función |
|:---|:---|:---|
| Sensor thread | 5 | Adquisición de datos de sensores (50 Hz) |
| Algorithm thread | 4 | Procesamiento y detección de crisis |
| Communication thread | 3 | Gestión de Bluetooth LE |
| Power management thread | 2 | Control de estados de energía |
| Watchdog thread | 1 | Supervisión del sistema |

---

## Módulos Principales

### main/
Punto de entrada del firmware. Inicializa el sistema, crea los hilos y gestiona el bucle principal de supervisión.

### drivers/
Controladores de bajo nivel para los periféricos:

| Módulo | Descripción |
|:---|:---|
| `ppg.c` | Driver para sensor óptico MAX86150 (frecuencia cardíaca, SpO₂) |
| `gsr.c` | Driver para sensor de conductancia dérmica AD5940 |
| `accelerometer.c` | Driver para acelerómetro LIS3DH |
| `temperature.c` | Driver para sensor de temperatura TMP117 |
| `piezo.c` | Driver para actuador piezoeléctrico PIEZO-455 |
| `battery.c` | Gestión de batería y fuel gauge |

### algorithm/
Algoritmos de procesamiento de señales y detección:

| Archivo | Descripción |
|:---|:---|
| `detector.c` | Detección de crisis en tiempo real |
| `adaptive.c` | Ajuste adaptativo de umbrales según historial |

### communication/
Comunicación y seguridad:

| Archivo | Descripción |
|:---|:---|
| `ble.c` | Gestión de Bluetooth LE (publicidad, conexiones, GATT) |
| `security.c` | Cifrado, autenticación, gestión de claves |

### power/
Gestión de energía y modos de suspensión.

### storage/
Almacenamiento no volátil (FRAM) de configuraciones y datos históricos.

### bootloader/
Actualización de firmware Over-the-Air (OTA) segura.

---

## Configuración del Proyecto

### Archivos Kconfig
Las opciones de configuración se definen en archivos `Kconfig`:

```
firmware/
├── Kconfig          # Configuración principal
├── app/
│   └── Kconfig      # Configuración de la aplicación
└── drivers/
    └── Kconfig      # Configuración de drivers
```

### Device Tree
La descripción del hardware se define en archivos Device Tree:

```
firmware/
└── boards/
    └── arm/
        └── memoria_fija_v2.dts
```

---

## Compilación y Programación

### Requisitos previos
```bash
# Instalar nRF Connect SDK (versión 2.8.0)
# https://developer.nordicsemi.com/nRF_Connect_SDK/doc/latest/nrf/installation.html

# Clonar el repositorio
git clone https://github.com/enriqueherbertag-lgtm/Memoria-Fija-v2.git
cd Memoria-Fija-v2/firmware
```

### Compilar
```bash
# Configurar entorno (ajustar ruta según instalación)
source /opt/nrfconnect/zephyr/zephyr-env.sh

# Compilar para dispositivo objetivo
west build -b memoria_fija_v2 app/

# Compilar con configuraciones específicas
west build -b memoria_fija_v2 app/ -- -DCONFIG_DEBUG=y
```

### Programar
```bash
# Programar vía J-Link
west flash

# Programar vía DFU (USB)
west flash --runner nrfjprog

# Programar vía Bluetooth (OTA)
python tools/ota_send.py --file build/zephyr/app_update.bin --address XX:XX:XX:XX:XX:XX
```

### Depurar
```bash
# Depuración con J-Link
west debug

# Consola RTT (logs en tiempo real)
JLinkRTTClient
```

---

## Pruebas

### Pruebas unitarias
```bash
# Compilar y ejecutar pruebas unitarias
west test -p unit tests/unit/

# Ejecutar pruebas con cobertura
west test -p unit tests/unit/ --coverage
```

### Pruebas de integración
```bash
# Ejecutar pruebas en hardware simulado
west test -p native_sim tests/integration/
```

### Hardware-in-the-loop
```bash
# Ejecutar pruebas en hardware real
python tests/hardware-in-loop/run_tests.py
```

---

## Normas de Estilo

### Código fuente
- Lenguaje: C11 (con extensiones POSIX)
- Estilo: MISRA C:2023 (adaptado para dispositivos médicos)
- Indentación: 4 espacios (no tabs)
- Límite de línea: 100 caracteres
- Comentarios: Doxygen (estilo JavaDoc)

### Ejemplo
```c
/**
 * @brief Detects an imminent crisis based on physiological markers.
 *
 * @param hr Current heart rate (bpm)
 * @param gsr Current galvanic skin response (µS)
 * @param activity Current activity level (0-100)
 * @return true if crisis is detected, false otherwise
 */
bool detect_crisis(uint16_t hr, uint16_t gsr, uint8_t activity)
{
    // Calculate risk score
    uint32_t risk = (hr * 25) + (gsr * 20) + (activity * 20);
    
    // Compare with adaptive threshold
    return risk > g_adaptive_threshold;
}
```

---

## Control de Versiones

### Etiquetado de versiones
- `vX.Y.Z`: Versiones estables (X: mayor, Y: menor, Z: parche)
- `vX.Y.Z-rcN`: Candidatos a release
- `vX.Y.Z-dev`: Desarrollo en curso

### Ramas principales
- `main`: Código estable, versiones liberadas
- `develop`: Integración de desarrollo
- `feature/*`: Nuevas funcionalidades
- `bugfix/*`: Correcciones de errores

---

## Documentación

La documentación completa del API se genera con Doxygen:

```bash
# Generar documentación
doxygen docs/Doxyfile

# La documentación se genera en docs/html/
# Abrir con navegador: docs/html/index.html
```

---

## Licencia

El firmware de Memoria Fija v2 está licenciado bajo **Apache 2.0 con restricción de uso comercial**. Ver archivo `LICENSE` en la raíz del proyecto.

---

## Contacto

Para consultas sobre el firmware:
- **Desarrollador principal**: Enrique Aguayo H.
- **Email**: eaguayo@migst.cl
- **GitHub**: [@enriqueherbertag-lgtm](https://github.com/enriqueherbertag-lgtm)
