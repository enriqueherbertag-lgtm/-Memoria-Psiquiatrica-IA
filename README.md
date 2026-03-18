# Memoria Fija v2.1 - Sistema de Neuromodulación Psiquiátrica Subcutánea

**Dispositivo implantable autónomo para la detección y prevención de crisis en pacientes con trastornos psiquiátricos graves, con conectividad dual (Bluetooth + LTE-M) y clasificador IA en la nube.**

---

##  Descripción General

Memoria Fija v2.1 es un sistema de neuromodulación subcutánea implantado en la región retroauricular. Monitoriza marcadores fisiológicos de crisis psiquiátrica (agitación, arousal autonómico) e interviene automáticamente en menos de 60 segundos mediante estimulación piezoeléctrica adaptativa.

Para pacientes ambulatorios, incorpora **conectividad celular LTE-M** y un **servidor de filtrado con IA** que clasifica el riesgo y solo notifica al médico cuando es clínicamente relevante.

| Métrica | Valor |
|:---|:---|
| Tiempo de respuesta | < 60 segundos |
| Reducción de rehospitalizaciones esperada | ≥ 40% |
| Vida útil del dispositivo | 5 años |
| Costo por unidad | USD 299 |
| Ahorro anual estimado por paciente | USD 12.400 |

---

##  Estructura del Repositorio

### Documentación General
- [`docs/problema-clinico.md`](docs/problema-clinico.md) - El problema que resuelve
- [`docs/solucion-tecnica.md`](docs/solucion-tecnica.md) - La solución técnica
- [`docs/estudio-animal.md`](docs/estudio-animal.md) - Validación preclínica
- [`docs/protocolo-clinico.md`](docs/protocolo-clinico.md) - Estudio en humanos
- [`docs/salvaguardas-eticas.md`](docs/salvaguardas-eticas.md) - Ética y seguridad
- [`docs/uso-ambulatorio.md`](docs/uso-ambulatorio.md) - Uso fuera del hospital

### Hardware
- [`hardware/especificaciones-tecnicas.md`](hardware/especificaciones-tecnicas.md) - Specs del dispositivo
- [`hardware/componentes.md`](hardware/componentes.md) - Lista de componentes
- [`hardware/biocompatibilidad.md`](hardware/biocompatibilidad.md) - Ensayos de biocompatibilidad

### Firmware
- [`firmware/README.md`](firmware/README.md) - Estructura del firmware
- `firmware/drivers/` - Controladores de sensores y actuadores
  - [`ppg.md`](firmware/drivers/ppg.md) - Sensor óptico MAX86150
  - [`gsr.md`](firmware/drivers/gsr.md) - Sensor de conductancia dérmica AD5940
  - [`accel.md`](firmware/drivers/accel.md) - Acelerómetro LIS3DH
  - [`temp.md`](firmware/drivers/temp.md) - Sensor de temperatura TMP117
  - [`piezo.md`](firmware/drivers/piezo.md) - Actuador piezoeléctrico PIEZO-455
  - [`battery.md`](firmware/drivers/battery.md) - Gestión de batería MEC225 + MAX77756
- `firmware/algorithm/` - Algoritmos de procesamiento
  - [`detector.md`](firmware/algorithm/detector.md) - Detección de crisis
  - [`adaptive.md`](firmware/algorithm/adaptive.md) - Umbrales adaptativos
- `firmware/communication/` - Comunicaciones
  - [`ble.md`](firmware/communication/ble.md) - Bluetooth LE
  - [`cellular.md`](firmware/communication/cellular.md) - LTE-M/NB-IoT
  - [`dual-connectivity.md`](firmware/communication/dual-connectivity.md) - Conmutación automática

### Software
- [`software/README.md`](software/README.md) - Visión general del software
- [`software/app-medico/README.md`](software/app-medico/README.md) - App para médicos
- [`software/app-paciente/README.md`](software/app-paciente/README.md) - App para pacientes/cuidadores
- [`software/dashboard-hospital/README.md`](software/dashboard-hospital/README.md) - Dashboard hospitalario
- [`software/api/README.md`](software/api/README.md) - API para integración

### Nube (Cloud)
- [`cloud/README.md`](cloud/README.md) - Visión general de la arquitectura cloud
- [`cloud/arquitectura-cloud.md`](cloud/arquitectura-cloud.md) - Diagrama y componentes
- [`cloud/iot-hub.md`](cloud/iot-hub.md) - Ingesta de datos
- [`cloud/motor-ia-clasificador.md`](cloud/motor-ia-clasificador.md) - IA que clasifica riesgos
- [`cloud/servidor-filtro.md`](cloud/servidor-filtro.md) - Servidor que filtra alertas
- [`cloud/sistema-mensajeria.md`](cloud/sistema-mensajeria.md) - Notificaciones a médicos
- [`cloud/fhir-integration.md`](cloud/fhir-integration.md) - Integración con sistemas de salud

### Pacientes Ambulatorios
- [`ambulatorio/README.md`](ambulatorio/README.md) - Visión general
- [`ambulatorio/protocolo-alta.md`](ambulatorio/protocolo-alta.md) - Cómo enviar paciente a casa
- [`ambulatorio/consentimiento-remoto.md`](ambulatorio/consentimiento-remoto.md) - Consentimiento informado
- [`ambulatorio/bateria-extendida.md`](ambulatorio/bateria-extendida.md) - Gestión de energía

### Investigación
- [`investigacion/README.md`](investigacion/README.md) - Papers y datasets
- [`investigacion/papers-referencia/`](investigacion/papers-referencia/) - Artículos clave
- [`investigacion/estudios-previos/`](investigacion/estudios-previos/) - Estudios relacionados
- [`investigacion/datasets/`](investigacion/datasets/) - Datos para entrenamiento

### Modelo de Negocio
- [`negocio/README.md`](negocio/README.md) - Visión comercial
- [`negocio/modelo-comercial.md`](negocio/modelo-comercial.md) - Licencias y precios
- [`negocio/proyeccion-financiera.md`](negocio/proyeccion-financiera.md) - ROI y escalabilidad
- [`negocio/ruta-regulatoria.md`](negocio/ruta-regulatoria.md) - FDA, ISP, ANMAT
- [`negocio/propuesta-inversores.md`](negocio/propuesta-inversores.md) - Deck para inversores

### Comunidad
- [`comunidad/CONTRIBUTING.md`](comunidad/CONTRIBUTING.md) - Cómo contribuir
- [`comunidad/CODE_OF_CONDUCT.md`](comunidad/CODE_OF_CONDUCT.md) - Código de conducta

---

## 🚀 Estado del Proyecto

| Componente | Estado |
|:---|:---|
| Concepto clínico | ✅ Completado |
| Especificaciones hardware | ✅ Completado |
| Firmware drivers | ✅ Completado |
| Algoritmos de detección | ✅ Completado |
| Conectividad dual | ✅ Completado |
| Clasificador IA | ✅ Completado |
| Servidor filtro | ✅ Completado |
| Protocolo ambulatorio | ✅ Completado |
| Software de usuario | ⏳ En desarrollo |
| Integración FHIR | ⏳ Pendiente |
| Modelo de negocio | ⏳ Pendiente |

---

## 📬 Contacto

**Enrique Aguayo H.**  
Investigador independiente, Mackiber Labs  
Contacto: eaguayo@migst.cl  
ORCID: 0009-0004-4615-6825  
GitHub: [@enriqueherbertag-lgtm](https://github.com/enriqueherbertag-lgtm)

---

## ⚖️ Licencia

El código y documentación de este repositorio están protegidos bajo **Apache 2.0 con restricción de uso comercial**. Para uso comercial se requiere acuerdo por escrito con el autor.

---

> *"Un paciente psiquiátrico no es peligroso. Está en peligro. Y necesita ayuda antes de que sea tarde."*
