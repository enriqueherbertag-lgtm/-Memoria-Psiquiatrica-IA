# Salvaguardas Éticas de Memoria Fija v2

## Principios Fundamentales

El desarrollo e implementación de Memoria Fija v2 se rige por los siguientes principios éticos:

1. **Autonomía del paciente**: El dispositivo nunca actúa en contra de la voluntad explícita del paciente.
2. **Beneficencia**: Todas las intervenciones buscan el bienestar del paciente.
3. **No maleficencia**: Se minimizan los riesgos y se maximiza la seguridad.
4. **Justicia**: El acceso al dispositivo debe ser equitativo y no discriminatorio.
5. **Transparencia**: Cada decisión del sistema es trazable y explicable.

---

## Salvaguardas Incorporadas en el Dispositivo

### 1. Consentimiento informado dinámico

| Característica | Descripción |
|:---|:---|
| **Consentimiento inicial** | Firma de consentimiento informado detallado antes de la implantación. |
| **Revocación en cualquier momento** | El paciente puede desactivar el dispositivo temporalmente (24 h) o solicitar su retiro definitivo. |
| **Consentimiento para funciones específicas** | El paciente puede aceptar o rechazar funciones particulares (ej: mensajes de voz del médico, estimulación automática). |
| **Registro de cambios** | Cualquier modificación en el consentimiento queda registrada en blockchain. |

### 2. Límites máximos programables

| Parámetro | Límite máximo | Quién lo define |
|:---|:---|:---|
| Intensidad de estimulación | 80 dB (vía ósea) | Psiquiatra tratante |
| Duración máxima de estimulación | 60 segundos | Psiquiatra tratante |
| Frecuencia diaria de intervenciones | 10 eventos | Psiquiatra tratante |
| Horario de mensajes de voz | 8:00 - 22:00 | Psiquiatra tratante (con excepciones programables) |
| Umbral de detección de crisis | Personalizado por paciente | Algoritmo + ajuste médico |

### 3. Apagado automático por seguridad

El dispositivo se desactiva automáticamente si detecta:

- Sobrecalentamiento (> 42°C en la superficie del implante).
- Mal funcionamiento del actuador piezoeléctrico.
- Batería crítica (últimas 24 horas de autonomía).
- Intento de acceso no autorizado (más de 3 fallos de autenticación).

### 4. Modo avión (desconexión voluntaria)

El paciente puede activar un **"modo avión"** de 24 horas que:
- Suspende toda comunicación inalámbrica.
- Detiene la estimulación automática.
- Mantiene solo el registro local de datos.
- Se desactiva automáticamente tras 24 h (o antes si el paciente lo decide).

### 5. Fallback manual

En caso de emergencia, el paciente o cuidador puede:
- Presionar el área del implante (3 veces en 5 segundos) para desactivar temporalmente la estimulación.
- Usar la app para poner el dispositivo en modo seguro.
- Acudir a urgencias para desactivación por personal médico.

---

## Salvaguardas en la Comunicación Médico-Paciente

### 1. Autenticación biométrica del médico

Solo médicos autorizados pueden enviar mensajes de voz. La autenticación incluye:
- Credenciales de acceso (usuario + contraseña).
- Token biométrico (huella o reconocimiento facial).
- Registro en blockchain de cada intento de acceso.

### 2. Mensajes pregrabados vs. en tiempo real

| Tipo | Descripción | Control |
|:---|:---|:---|
| Mensajes pregrabados | Frases estandarizadas ("Respira", "Estoy contigo", "Toma tu medicación") | Aprobadas por el paciente durante el consentimiento. |
| Mensajes personalizados | Grabados por el médico en tiempo real | Solo en crisis, con registro obligatorio. |

### 3. Límites de comunicación

- Máximo 5 mensajes por hora.
- Máximo 20 mensajes por día.
- Duración máxima por mensaje: 15 segundos.
- Solo emisor autorizado (el psiquiatra tratante).

### 4. Registro y trazabilidad

Cada mensaje enviado queda registrado con:
- Fecha y hora.
- Contenido (transcrito).
- Emisor (médico autenticado).
- Receptor (paciente).
- Respuesta fisiológica del paciente (datos del implante).

---

## Salvaguardas en el Algoritmo de Detección

### 1. Umbrales conservadores por defecto

Inicialmente, los umbrales de detección se configuran en modo **"alta sensibilidad, baja especificidad"** para evitar falsos negativos (crisis no detectadas). Durante los primeros 3 meses, el médico ajusta los umbrales según la evolución del paciente.

### 2. Doble verificación humana para crisis graves

Si el algoritmo detecta una crisis inminente con alta probabilidad (> 90%):
1. Se activa la estimulación preventiva.
2. Se notifica al médico vía app.
3. El médico puede confirmar o anular la intervención en los siguientes 60 segundos.
4. Si no hay respuesta del médico, el dispositivo procede con la intervención automática.

### 3. Aprendizaje continuo supervisado

El algoritmo se actualiza mensualmente con:
- Datos del paciente.
- Revisión médica de eventos.
- Ajustes de umbrales.
- Supervisión del Comité de Ética.

### 4. Explicabilidad

Cada decisión del algoritmo puede ser explicada:
- "Se detectó crisis porque la frecuencia cardíaca aumentó un 40% en 5 minutos."
- "Se activó estimulación porque el puntaje de riesgo superó el umbral personalizado (85/100)."
- "No se intervino porque el paciente activó modo avión."

---

## Salvaguardas en el Manejo de Datos

### 1. Propiedad de los datos

| Tipo de dato | Propietario | Acceso |
|:---|:---|:---|
| Datos fisiológicos crudos | Paciente | Solo el paciente (vía app) |
| Registros de crisis | Paciente + médico tratante | Compartido con consentimiento |
| Estadísticas anonimizadas | Investigación | Uso agregado, sin identificación |
| Registros en blockchain | Público (hash) | Solo verificación, no datos sensibles |

### 2. Cifrado y seguridad

- Datos en reposo: cifrado AES-256.
- Datos en tránsito: TLS 1.3 + autenticación mutua.
- Almacenamiento en blockchain: solo hashes, no datos personales.
- Acceso a la app: autenticación biométrica + PIN.

### 3. Política de retención

- Datos fisiológicos: almacenados 5 años (vida útil del dispositivo).
- Registros de crisis: almacenados 10 años (requisito legal).
- Datos anonimizados para investigación: indefinido, con consentimiento.

### 4. Derecho al olvido

El paciente puede solicitar:
- Eliminación de todos sus datos.
- Retiro del dispositivo.
- Exclusión de futuras investigaciones.
- Objeción a nuevas funcionalidades.

---

## Comités de Supervisión

### 1. Comité de Ética Independiente

- Revisa el protocolo de investigación.
- Monitorea eventos adversos.
- Audita el cumplimiento de salvaguardas.
- Emite informes públicos anuales.

### 2. Comité de Monitoreo de Datos y Seguridad (DSMB)

- Revisa datos de seguridad cada 3 meses.
- Puede detener el estudio si se detectan riesgos inaceptables.
- Compuesto por expertos independientes (psiquiatras, bioeticistas, estadísticos).

### 3. Comité de Pacientes y Familiares

- Participa en el diseño de nuevas funcionalidades.
- Revisa materiales informativos.
- Aporta perspectiva de usuarios reales.
- Vela por la accesibilidad y no discriminación.

---

## Consideraciones Especiales

### Pacientes con capacidad disminuida
- Evaluación de capacidad por psiquiatra independiente.
- Consentimiento con apoyo familiar.
- Defensor del paciente designado.
- Revisión periódica de la voluntad del paciente.

### Uso en poblaciones vulnerables
- No se incluirán pacientes institucionalizados sin consentimiento explícito.
- Especial protección para mujeres embarazadas.
- Exclusión de menores de edad en fase inicial.

### Acceso equitativo
- Los dispositivos se asignarán por sorteo entre candidatos elegibles.
- Se garantizará acceso a pacientes de bajos recursos.
- Se establecerá un fondo de becas para quienes no puedan costearlo.

---

## Referencias

1. Declaración de Helsinki (2013). "Principios éticos para las investigaciones médicas en seres humanos".
2. UNESCO. (2021). "Declaración universal sobre bioética y derechos humanos".
3. Nuffield Council on Bioethics. (2022). "Neurotechnology and human rights".
4. IEEE Global Initiative on Ethics of Autonomous and Intelligent Systems. (2024). "Ethically Aligned Design".
5. World Medical Association. (2023). "Statement on ethical aspects of implantable neurotechnologies".
6. Comisión Nacional de Bioética de Chile. (2024). "Guía para la investigación en neurotecnologías".
