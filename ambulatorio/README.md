# Pacientes Ambulatorios - Memoria Fija v2.1

## Visión General

Esta sección documenta todo lo relacionado con el uso del dispositivo en **pacientes ambulatorios** (aquellos que han sido dados de alta del hospital y continúan su tratamiento en domicilio).

---

## Componentes del Sistema Ambulatorio

```
┌─────────────────────────────────────────────────────────────┐
│                      PACIENTE EN DOMICILIO                   │
│                                                               │
│  [Dispositivo implantado con LTE-M]                          │
│  [Cargador inalámbrico]                                      │
│  [App cuidador (opcional)]                                   │
└─────────────────────────────────────────────────────────────┘
                            ↓ (datos cada hora)
┌─────────────────────────────────────────────────────────────┐
│                      SERVIDOR CLOUD                          │
│  (recepción, clasificación, filtrado)                       │
└─────────────────────────────────────────────────────────────┘
                            ↓ (solo alertas)
┌─────────────────────────────────────────────────────────────┐
│                      MÉDICO TRATANTE                         │
│  (recibe notificaciones cuando es necesario)                │
└─────────────────────────────────────────────────────────────┘
```

---

## Documentación Disponible

| Archivo | Descripción |
|:---|:---|
| [`protocolo-alta.md`](protocolo-alta.md) | Procedimiento para dar de alta a un paciente |
| [`consentimiento-remoto.md`](consentimiento-remoto.md) | Consentimiento informado para telemonitoreo |
| [`bateria-extendida.md`](bateria-extendida.md) | Gestión de energía para uso domiciliario |
| [`kit-alta.md`](kit-alta.md) | Elementos entregados al paciente |

---

## Flujo de Operación Ambulatoria

### Día del alta
1. Verificar criterios de elegibilidad
2. Configurar dispositivo en modo ambulatorio
3. Activar módulo LTE-M
4. Entregar kit al paciente
5. Firmar consentimiento remoto
6. Primera transmisión de prueba

### Seguimiento diario
- Dispositivo transmite datos cada hora vía LTE-M
- Servidor procesa y clasifica automáticamente
- Solo se notifica al médico si hay anomalías

### Eventos especiales
- **Batería baja**: Notificación al cuidador
- **Sin datos > 24h**: Alerta por posible fallo
- **Crisis inminente**: Alerta inmediata a médico + emergencias

---

## Ventajas del Modelo Ambulatorio

| Aspecto | Beneficio |
|:---|:---|
| **Calidad de vida** | Paciente en su hogar, no hospitalizado |
| **Costo** | Reducción del 80% vs hospitalización |
| **Detección temprana** | Monitorización 24/7 |
| **Tranquilidad familiar** | Saben que el paciente está monitoreado |
| **Datos para el médico** | Información continua, no solo en crisis |

---

## Métricas Clave

| Métrica | Objetivo |
|:---|:---|
| Tasa de transmisión exitosa | > 99% |
| Tiempo medio sin datos | < 2 horas |
| Alertas gestionadas sin reingreso | > 90% |
| Satisfacción del paciente | > 8.5/10 |
