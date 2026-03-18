# Integración FHIR - Memoria Fija Cloud

## Descripción

La integración con el estándar **HL7 FHIR (Fast Healthcare Interoperability Resources)** permite que Memoria Fija se conecte con los sistemas de historia clínica electrónica (EHR) de hospitales y clínicas, enviando y recibiendo información de forma estandarizada.

---

## Recursos FHIR Utilizados

| Recurso | Uso | Datos intercambiados |
|:---|:---|:---|
| **Patient** | Identificación del paciente | Nombre, ID, datos demográficos |
| **Observation** | Resultados de monitoreo | HR, GSR, actividad, temperatura |
| **Condition** | Diagnósticos | Trastorno psiquiátrico |
| **RiskAssessment** | Nivel de riesgo | Probabilidad de crisis |
| **Communication** | Alertas y mensajes | Notificaciones al médico |
| **Device** | Información del implante | Estado, batería, firmware |
| **CarePlan** | Plan de tratamiento | Protocolos personalizados |

---

## Arquitectura de Integración

```
[MEMORIA FIJA CLOUD]
    ↓ (eventos clasificados)
[FHIR CONVERTER]
    ↓ (recursos FHIR)
[FHIR SERVER]
    ↓ (API RESTful)
[HOSPITAL EHR]
    ↓
[Médico visualiza en su sistema habitual]
```

---

## Ejemplo: Recurso Observation (Datos de paciente)

```json
{
  "resourceType": "Observation",
  "id": "obs-123456",
  "status": "final",
  "category": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/observation-category",
      "code": "vital-signs",
      "display": "Vital Signs"
    }]
  }],
  "code": {
    "coding": [{
      "system": "http://loinc.org",
      "code": "8867-4",
      "display": "Heart rate"
    }]
  },
  "subject": {
    "reference": "Patient/789012"
  },
  "effectiveDateTime": "2026-03-18T10:30:00Z",
  "valueQuantity": {
    "value": 75,
    "unit": "beats/minute",
    "system": "http://unitsofmeasure.org",
    "code": "/min"
  },
  "device": {
    "reference": "Device/memoria-fija-001"
  }
}
```

---

## Ejemplo: Recurso RiskAssessment (Riesgo de crisis)

```json
{
  "resourceType": "RiskAssessment",
  "id": "risk-123456",
  "status": "final",
  "subject": {
    "reference": "Patient/789012"
  },
  "occurrenceDateTime": "2026-03-18T10:30:00Z",
  "basis": [{
    "reference": "Observation/obs-123456"
  }],
  "prediction": [{
    "outcome": {
      "coding": [{
        "system": "https://memoriafija.cl/CodeSystem/crisis-types",
        "code": "PSYCHIATRIC_CRISIS",
        "display": "Crisis psiquiátrica"
      }]
    },
    "probabilityDecimal": 0.92,
    "whenRange": {
      "low": {
        "value": 0,
        "unit": "minutes"
      },
      "high": {
        "value": 60,
        "unit": "minutes"
      }
    }
  }],
  "mitigation": "Activar protocolo de crisis, notificar a médico",
  "note": [{
    "text": "Riesgo alto detectado por IA (probabilidad 92%)"
  }]
}
```

---

## Ejemplo: Recurso Communication (Alerta al médico)

```json
{
  "resourceType": "Communication",
  "id": "comm-123456",
  "status": "completed",
  "category": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/communication-category",
      "code": "alert",
      "display": "Alert"
    }]
  }],
  "priority": "stat",
  "subject": {
    "reference": "Patient/789012"
  },
  "about": [{
    "reference": "RiskAssessment/risk-123456"
  }],
  "sent": "2026-03-18T10:30:05Z",
  "recipient": [{
    "reference": "Practitioner/456789"
  }],
  "sender": {
    "reference": "Device/memoria-fija-cloud-001"
  },
  "payload": [{
    "contentString": "Alerta de crisis inminente para paciente Juan Pérez. Probabilidad 92%. Activar protocolo de emergencia."
  }],
  "reasonCode": [{
    "coding": [{
      "system": "http://terminology.hl7.org/CodeSystem/v3-ActReason",
      "code": "EMERG",
      "display": "Emergency"
    }]
  }]
}
```

---

## Implementación en Python

```python
import requests
import json
from datetime import datetime

class FHIRClient:
    def __init__(self, base_url, access_token):
        self.base_url = base_url
        self.headers = {
            'Authorization': f'Bearer {access_token}',
            'Content-Type': 'application/fhir+json'
        }
    
    def create_observation(self, patient_id, codigo_loinc, valor, unidad):
        """Crea una observación FHIR."""
        
        observation = {
            "resourceType": "Observation",
            "status": "final",
            "category": [{
                "coding": [{
                    "system": "http://terminology.hl7.org/CodeSystem/observation-category",
                    "code": "vital-signs",
                    "display": "Vital Signs"
                }]
            }],
            "code": {
                "coding": [{
                    "system": "http://loinc.org",
                    "code": codigo_loinc,
                    "display": "Heart rate"
                }]
            },
            "subject": {
                "reference": f"Patient/{patient_id}"
            },
            "effectiveDateTime": datetime.now().isoformat(),
            "valueQuantity": {
                "value": valor,
                "unit": unidad,
                "system": "http://unitsofmeasure.org",
                "code": unidad
            }
        }
        
        response = requests.post(
            f"{self.base_url}/Observation",
            json=observation,
            headers=self.headers
        )
        
        return response.json()
    
    def create_risk_assessment(self, patient_id, observaciones, probabilidad, notas):
        """Crea una evaluación de riesgo FHIR."""
        
        risk = {
            "resourceType": "RiskAssessment",
            "status": "final",
            "subject": {
                "reference": f"Patient/{patient_id}"
            },
            "occurrenceDateTime": datetime.now().isoformat(),
            "basis": [{"reference": f"Observation/{obs}"} for obs in observaciones],
            "prediction": [{
                "outcome": {
                    "coding": [{
                        "system": "https://memoriafija.cl/CodeSystem/crisis-types",
                        "code": "PSYCHIATRIC_CRISIS",
                        "display": "Crisis psiquiátrica"
                    }]
                },
                "probabilityDecimal": probabilidad,
                "whenRange": {
                    "low": {"value": 0, "unit": "minutes"},
                    "high": {"value": 60, "unit": "minutes"}
                }
            }],
            "note": [{"text": notas}]
        }
        
        response = requests.post(
            f"{self.base_url}/RiskAssessment",
            json=risk,
            headers=self.headers
        )
        
        return response.json()
```

---

## Mapeo de Códigos LOINC

| Variable | Código LOINC | Display |
|:---|:---|:---|
| Frecuencia cardíaca | 8867-4 | Heart rate |
| Variabilidad cardíaca | 80404-7 | Heart rate variability |
| Conductancia dérmica | 44657-9 | Skin conductance |
| Temperatura corporal | 8310-5 | Body temperature |
| Actividad física | 73963-3 | Physical activity |
| Riesgo psiquiátrico | 95209-3 | Psychiatric crisis risk |

---

## Ventajas de Usar FHIR

| Beneficio | Descripción |
|:---|:---|
| **Estandarización** | Cualquier hospital que cumpla FHIR puede integrarse |
| **Futuro-proof** | Evoluciona con el estándar global |
| **Seguridad** | Soporte nativo para autenticación OAuth2 |
| **Eficiencia** | Recursos optimizados para intercambio de datos |
| **Adopción** | Usado por Epic, Cerner, y principales EHR |

---

## Estado de Implementación

| Componente | Estado |
|:---|:---|
| Mapeo de recursos | ✅ Completado |
| Cliente Python | ✅ Completado |
| Autenticación OAuth2 | ⏳ Pendiente |
| Pruebas con EHR real | ⏳ Pendiente |
| Documentación API | ⏳ Pendiente |
