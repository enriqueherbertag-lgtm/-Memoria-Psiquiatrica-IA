# Componentes de Memoria Fija v2

## Listado de Componentes Principales

### Microcontrolador y Comunicación
| Componente | Modelo | Fabricante | Función |
|:---|:---|:---|:---|
| SoC principal | nRF5340 | Nordic Semiconductor | Procesador dual-core + Bluetooth LE |
| Memoria FRAM | MB85RS2MT | Fujitsu | Almacenamiento no volátil de datos |
| Cristal 32 MHz | NX3225GD | NDK | Reloj principal |
| Cristal 32.768 kHz | NX1210AB | NDK | Reloj de tiempo real |

### Sensores
| Componente | Modelo | Fabricante | Función |
|:---|:---|:---|:---|
| Sensor PPG | MAX86150 | Maxim Integrated | Frecuencia cardíaca, SpO₂ |
| Sensor GSR | AD5940 | Analog Devices | Conductancia dérmica |
| Acelerómetro | LIS3DH | STMicroelectronics | Actividad motora, agitación |
| Sensor temperatura | TMP117 | Texas Instruments | Temperatura corporal |

### Actuador
| Componente | Modelo | Fabricante | Función |
|:---|:---|:---|:---|
| Actuador piezoeléctrico | PIEZO-455 | Mide Technology | Estimulación + mensajes voz |
| Driver piezoeléctrico | DRV2667 | Texas Instruments | Amplificador de alto voltaje |

### Sistema de Alimentación
| Componente | Modelo | Fabricante | Función |
|:---|:---|:---|:---|
| Batería | MEC225 | EaglePicher | Almacenamiento energía (200 mAh) |
| PMIC | MAX77756 | Maxim Integrated | Gestión de carga y alimentación |
| Bobina carga Qi | 760308100110 | Würth Elektronik | Recepción carga inalámbrica |
| Regulador LDO | TPS7A02 | Texas Instruments | Regulación 3.3V (bajo consumo) |

### Pasivos y Protecciones
| Componente | Especificación | Función |
|:---|:---|:---|
| Capacitores cerámicos | 0402, X7R | Filtrado y desacoplo |
| Capacitores de tantalio | 0603, 10 µF | Almacenamiento local |
| Inductores | 0402, 1 µH | Filtros conmutados |
| Diodos ESD | PESD5V0S1UB | Protección contra descargas |
| Fusible rearmable | 0.5A, 6V | Protección sobrecorriente |

### Conectividad (Gateway)
| Componente | Modelo | Fabricante | Función |
|:---|:---|:---|:---|
| Tablet industrial | TC8300 | Zebra Technologies | Gateway hospitalario |
| Adaptador Bluetooth | USB BT500 | Sena | Extensor alcance (opcional) |
| Router 4G/5G | RUTX11 | Teltonika | Respaldo conectividad |

---

## Proveedores Calificados

### Microcontroladores y Semiconductores
| Fabricante | Productos | Certificaciones |
|:---|:---|:---|
| Nordic Semiconductor | nRF53 series | ISO 13485 (dispositivos médicos) |
| Texas Instruments | PMIC, sensores, drivers | ISO 13485, IATF 16949 |
| Analog Devices | AD5940, amplificadores | ISO 13485, AS9100D |
| STMicroelectronics | MEMS, acelerómetros | ISO 13485, ISO 26262 |
| Maxim Integrated | PPG, PMIC | ISO 13485, ISO 9001 |

### Baterías
| Fabricante | Modelo | Certificaciones |
|:---|:---|:---|
| EaglePicher | MEC225 | ISO 13485, UL 1642 |
| Saft | LS 14500 | ISO 13485, IEC 60086 |
| Tadiran | TL-5902 | ISO 13485, UL 913 |

### Materiales para Carcasa
| Proveedor | Material | Certificaciones |
|:---|:---|:---|
| Carpenter Technology | Ti-6Al-4V ELI | ASTM F136, ISO 5832-3 |
| Sandvik | Titanio grado 23 | ASTM F136, ISO 5832-3 |
| NuSil | Silicona médica | ISO 10993, USP Clase VI |

---

## Especificaciones de Compra

### Cantidades mínimas
| Componente | MOQ (muestreo) | MOQ (producción) |
|:---|:---|:---|
| nRF5340 | 10 unidades | 1000 unidades |
| MAX86150 | 10 unidades | 500 unidades |
| AD5940 | 5 unidades | 250 unidades |
| PIEZO-455 | 5 unidades | 100 unidades |
| MEC225 | 2 unidades | 50 unidades |

### Tiempos de entrega
| Componente | Tiempo (muestreo) | Tiempo (producción) |
|:---|:---|:---|
| Semiconductores estándar | 2-4 semanas | 8-12 semanas |
| Semiconductores especializados | 4-6 semanas | 12-16 semanas |
| Baterías | 2-3 semanas | 6-8 semanas |
| Materiales titanio | 3-4 semanas | 4-6 semanas |

### Precios orientativos (USD)
| Componente | Precio unitario (10 uds) | Precio unitario (1000 uds) |
|:---|:---|:---|
| nRF5340 | 8.50 | 4.20 |
| MAX86150 | 6.20 | 3.10 |
| AD5940 | 9.80 | 4.90 |
| LIS3DH | 1.80 | 0.85 |
| TMP117 | 2.10 | 1.05 |
| PIEZO-455 | 15.00 | 7.50 |
| MEC225 | 25.00 | 12.50 |
| PCB (fabricación) | 50.00 (prototipo) | 5.00 (volumen) |

---

## Alternativas y Segundas Fuentes

### Microcontrolador
| Alternativa | Ventajas | Desventajas |
|:---|:---|:---|
| nRF52840 | Más económico | Menor rendimiento, sin segundo core |
| STM32WB55 | Mejor soporte ST | Mayor consumo |
| EFR32BG22 | Ultra bajo consumo | Menor memoria |

### Sensor PPG
| Alternativa | Ventajas | Desventajas |
|:---|:---|:---|
| AFE4900 | Integra ECG | Mayor complejidad |
| MAX30102 | Más común | Menor SNR |
| BH1790GLC | Bajo consumo | Solo verde |

### Batería
| Alternativa | Ventajas | Desventajas |
|:---|:---|:---|
| Saft LS 14500 | Mayor capacidad | Mayor tamaño |
| Tadiran TL-5902 | Larga vida útil | Mayor costo |
| Panasonic BR1632A | Formato botón | Menor capacidad |

---

## Componentes para Prototipado

### Kit de desarrollo
| Componente | Modelo | Proveedor |
|:---|:---|:---|
| Placa evaluación nRF53 | nRF5340 DK | Nordic Semiconductor |
| Placa evaluación PPG | MAX86150EVSYS | Maxim Integrated |
| Placa evaluación GSR | EVAL-AD5940 | Analog Devices |
| Actuador piezo | PIEZO-455 (con cable) | Mide Technology |
| Batería prototipo | MEC225 (con conector) | EaglePicher |

### Módulos para pruebas
| Módulo | Función | Proveedor |
|:---|:---|:---|
| Bluetooth USB dongle | nRF52840 Dongle | Nordic Semiconductor |
| Analizador de baterías | Monsoon HV | Monsoon Solutions |
| Analizador de protocolos | Ellisys Bluetooth Vanguard | Ellisys |
| Cámara anecoica | RF-shielded box | Ramsey Electronics |

---

## Gestión de Inventario

### Códigos de parte internos
| Componente | Código Mackiber |
|:---|:---|
| nRF5340 | MF-CPU-001 |
| MAX86150 | MF-SEN-001 |
| AD5940 | MF-SEN-002 |
| LIS3DH | MF-SEN-003 |
| TMP117 | MF-SEN-004 |
| PIEZO-455 | MF-ACT-001 |
| MEC225 | MF-BAT-001 |

### Trazabilidad
- Cada lote de componentes tiene número de lote trazable.
- Se mantiene registro de fecha de compra, proveedor y certificados.
- Los componentes críticos tienen análisis de lote (muestreo).
- Se almacenan en condiciones controladas (temperatura, humedad).

---

## Referencias

1. ISO 13485:2022. "Medical devices - Quality management systems".
2. IPC-7351B. "Generic Requirements for Surface Mount Design and Land Pattern Standard".
3. JEDEC J-STD-020. "Moisture/Reflow Sensitivity Classification for Nonhermetic Surface Mount Devices".
4. UL 1642. "Standard for Lithium Batteries".
5. IEC 62133-2:2023. "Secondary cells and batteries containing alkaline or other non-acid electrolytes".
6. Digi-Key. (2026). "Component selection guide for medical devices".
7. Mouser Electronics. (2026). "Medical electronics buyer's guide".
