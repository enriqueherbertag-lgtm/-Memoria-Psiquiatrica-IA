# Sistema de Mensajería - Memoria Fija Cloud

## Descripción

El sistema de mensajería gestiona el envío de alertas y notificaciones desde el servidor a los médicos, cuidadores y servicios de emergencia, garantizando que la información llegue por el canal adecuado según la urgencia.

---

## Arquitectura de Mensajería

```
[MOTOR DE CLASIFICACIÓN]
    ↓ (evento clasificado)
[SERVICIO DE MENSAJERÍA]
    ↓
┌────────────────────────────────────┐
│    DETERMINAR DESTINATARIOS        │
│  - Médico tratante                  │
│  - Cuidador principal               │
│  - Emergencias (si aplica)          │
└────────────────────────────────────┘
    ↓
┌────────────────────────────────────┐
│    CONSULTAR PREFERENCIAS           │
│  - App push                         │
│  - SMS                               │
│  - Email                             │
│  - Llamada automática                │
└────────────────────────────────────┘
    ↓
┌────────────────────────────────────┐
│    ENVÍO MULTICANAL                  │
│  - Enviar por todos los canales      │
│  - Esperar confirmación               │
│  - Escalar si no hay respuesta        │
└────────────────────────────────────┘
```

---

## Tipos de Mensajes

| Tipo | Prioridad | Canales | Escalamiento |
|:---|:---|:---|:---|
| **CRISIS INMINENTE** | 1 (máxima) | App + SMS + Emergencias | Inmediato a 911 |
| **ALERTA ALTA** | 2 | App + SMS | Escalar a otro médico si no responde en 15 min |
| **ALERTA MODERADA** | 3 | App + Email | Recordatorio diario |
| **INFORMATIVO** | 4 | App | Sin escalamiento |

---

## Estructura de Datos

```python
class Mensaje:
    def __init__(self, paciente_id, tipo, contenido):
        self.id = generar_uuid()
        self.timestamp = datetime.now()
        self.paciente_id = paciente_id
        self.tipo = tipo  # 'CRISIS', 'ALTA', 'MODERADA', 'INFO'
        self.contenido = contenido
        self.destinatarios = []
        self.estado = 'PENDIENTE'
        self.confirmaciones = []
        self.escalado = False

class Destinatario:
    def __init__(self, tipo, id, preferencias):
        self.tipo = tipo  # 'MEDICO', 'CUIDADOR', 'EMERGENCIAS'
        self.id = id
        self.preferencias = preferencias  # {'push': True, 'sms': True, 'email': False}
        self.contacto_push = None
        self.contacto_sms = None
        self.contacto_email = None
```

---

## Lógica de Envío

```python
import time
import requests
from datetime import datetime, timedelta

class ServicioMensajeria:
    def __init__(self):
        self.canales = {
            'push': CanalPush(),
            'sms': CanalSMS(),
            'email': CanalEmail(),
            'llamada': CanalLlamada()
        }
    
    def enviar_alerta(self, paciente_id, nivel_riesgo, datos):
        # 1. Obtener información del paciente
        paciente = self.obtener_paciente(paciente_id)
        
        # 2. Determinar destinatarios
        destinatarios = self.determinar_destinatarios(paciente, nivel_riesgo)
        
        # 3. Generar contenido del mensaje
        contenido = self.generar_contenido(paciente, nivel_riesgo, datos)
        
        # 4. Enviar a cada destinatario por sus canales preferidos
        for dest in destinatarios:
            self.enviar_a_destinatario(dest, contenido, nivel_riesgo)
        
        # 5. Registrar envío
        self.registrar_envio(paciente_id, nivel_riesgo, destinatarios)
        
        # 6. Iniciar monitoreo de confirmaciones
        self.monitorear_confirmaciones(paciente_id, nivel_riesgo)
    
    def determinar_destinatarios(self, paciente, nivel_riesgo):
        destinatarios = []
        
        # Siempre incluir al médico tratante
        destinatarios.append(paciente.medico)
        
        # Incluir cuidador si está configurado
        if paciente.cuidador:
            destinatarios.append(paciente.cuidador)
        
        # Incluir emergencias si es crisis
        if nivel_riesgo == 'CRITICO':
            destinatarios.append(self.get_emergencias(paciente.ubicacion))
        
        return destinatarios
    
    def enviar_a_destinatario(self, destinatario, contenido, nivel_riesgo):
        # Enviar por todos los canales preferidos del destinatario
        for canal, habilitado in destinatario.preferencias.items():
            if habilitado and canal in self.canales:
                self.canales[canal].enviar(
                    destinatario.contacto(canal),
                    contenido,
                    nivel_riesgo
                )
        
        # Registrar intento de envío
        self.registrar_intento(destinatario.id, contenido)
    
    def monitorear_confirmaciones(self, paciente_id, nivel_riesgo):
        """Espera confirmaciones y escala si no hay respuesta."""
        
        tiempo_espera = {
            'CRITICO': 60,      # 1 minuto
            'ALTO': 900,        # 15 minutos
            'MODERADO': 3600,    # 1 hora
            'INFO': 86400        # 24 horas
        }
        
        time.sleep(tiempo_espera[nivel_riesgo])
        
        if not self.hay_confirmacion(paciente_id):
            self.escalar_alerta(paciente_id, nivel_riesgo)
    
    def escalar_alerta(self, paciente_id, nivel_riesgo):
        """Escala la alerta a un nivel superior."""
        
        if nivel_riesgo == 'ALTO':
            # Escalar a crisis
            self.enviar_alerta(paciente_id, 'CRITICO', {'escalado': True})
        
        elif nivel_riesgo == 'MODERADO':
            # Escalar a alto
            self.enviar_alerta(paciente_id, 'ALTO', {'escalado': True})
        
        # Registrar escalamiento
        self.registrar_escalamiento(paciente_id, nivel_riesgo)
```

---

## Canales de Comunicación

### Canal Push (App Médico)

```python
class CanalPush:
    def enviar(self, token, contenido, prioridad):
        # Usar Firebase Cloud Messaging o similar
        payload = {
            'to': token,
            'priority': 'high' if prioridad == 'CRITICO' else 'normal',
            'notification': {
                'title': f'ALERTA {prioridad}',
                'body': contenido.resumen,
                'sound': 'default',
                'click_action': 'OPEN_PATIENT'
            },
            'data': contenido.data
        }
        
        response = requests.post(
            'https://fcm.googleapis.com/fcm/send',
            json=payload,
            headers={'Authorization': f'key={FCM_SERVER_KEY}'}
        )
        
        return response.status_code == 200
```

### Canal SMS

```python
class CanalSMS:
    def enviar(self, telefono, contenido, prioridad):
        # Usar servicio de SMS (Twilio, AWS SNS, etc.)
        mensaje = f"[MEMORIA FIJA] {contenido.resumen}"
        
        if prioridad == 'CRITICO':
            mensaje = f"🚨 URGENTE: {mensaje}"
        
        # Enviar SMS
        response = requests.post(
            SMS_API_URL,
            json={
                'to': telefono,
                'message': mensaje
            },
            headers={'Authorization': f'Bearer {SMS_API_KEY}'}
        )
        
        return response.status_code == 200
```

### Canal Email

```python
class CanalEmail:
    def enviar(self, email, contenido, prioridad):
        # Usar servicio de email (SendGrid, AWS SES)
        asunto = f"[MEMORIA FIJA] Alerta {prioridad}"
        
        if prioridad == 'CRITICO':
            asunto = f"🚨 URGENTE: {asunto}"
        
        # Enviar email
        response = requests.post(
            EMAIL_API_URL,
            json={
                'to': email,
                'subject': asunto,
                'html': contenido.html,
                'text': contenido.texto
            },
            headers={'Authorization': f'Bearer {EMAIL_API_KEY}'}
        )
        
        return response.status_code == 200
```

---

## Confirmaciones

```python
def registrar_confirmacion(medico_id, paciente_id):
    """Registra que un médico confirmó haber recibido la alerta."""
    
    confirmacion = {
        'medico_id': medico_id,
        'paciente_id': paciente_id,
        'timestamp': datetime.now(),
        'tipo': 'CONFIRMACION'
    }
    
    # Guardar en base de datos
    db.insert('confirmaciones', confirmacion)
    
    # Actualizar estado de la alerta
    actualizar_estado_alerta(paciente_id, 'CONFIRMADA')
    
    return True
```

---

## Métricas y Monitoreo

| Métrica | Descripción | Objetivo |
|:---|:---|:---|
| Tiempo de entrega push | Latencia notificación | < 5 seg |
| Tasa de apertura | % de alertas vistas | > 90% |
| Tiempo de respuesta | Tiempo hasta confirmación | < 2 min (crítico) |
| Escalamientos | % que requieren escalado | < 5% |
