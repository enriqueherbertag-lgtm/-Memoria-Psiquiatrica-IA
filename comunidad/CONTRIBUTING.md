# Guía de Contribución - Memoria Fija v2.1

## Formas de Contribuir

### Reportar problemas (Issues)

Si encuentras un error o tienes una sugerencia:

1. Busca primero en los issues existentes para evitar duplicados
2. Usa la plantilla correspondiente (bug report, feature request)
3. Incluye información detallada:
   - Versión del firmware/software
   - Pasos para reproducir el error
   - Logs o mensajes de error
   - Configuración del dispositivo

### Sugerir mejoras (Feature requests)

- Describe claramente la funcionalidad propuesta
- Explica por qué sería útil para pacientes o médicos
- Si es posible, menciona cómo podría implementarse
- Incluye referencias a estudios o proyectos similares

### Contribuir con codigo (Pull Requests)

1. Fork del repositorio
2. Crear una rama con nombre descriptivo
3. Seguir los estándares de código
4. Incluir pruebas cuando corresponda
5. Actualizar documentación si es necesario
6. Hacer pull request a la rama `develop`

---

## Estándares de Código

### Firmware (C)

- Seguir MISRA C:2023 (adaptado para dispositivos médicos)
- Usar nombres descriptivos (variables, funciones, constantes)
- Incluir comentarios Doxygen
- Mantener funciones pequeñas (< 100 líneas)
- Documentar secciones complejas

```c
/**
 * @brief Calcula el nivel de riesgo basado en sensores.
 *
 * @param hr Frecuencia cardíaca (bpm)
 * @param gsr Conductancia dérmica (µS)
 * @param activity Nivel de actividad (0-100)
 * @return Nivel de riesgo (0-100)
 */
uint8_t calculate_risk(uint16_t hr, uint16_t gsr, uint8_t activity)
{
    // Validación de rangos
    if (hr > 200 || hr < 30) return 100;
    
    // Cálculo
    uint8_t risk = (hr / 2) + (gsr / 10) + activity;
    
    return risk > 100 ? 100 : risk;
}
```

### Python (Cloud/IA)

- Seguir PEP 8
- Usar type hints
- Incluir docstrings (formato Google o NumPy)
- Mantener funciones enfocadas en una sola tarea

```python
def classify_risk(patient_data: dict) -> dict:
    """
    Clasifica el nivel de riesgo de un paciente.
    
    Args:
        patient_data: Diccionario con datos del paciente
        
    Returns:
        Diccionario con nivel de riesgo y acciones recomendadas
    """
    # Implementación
    return risk_level
```

### Documentación (Markdown)

- Usar títulos jerárquicos (`#`, `##`, `###`)
- Separar secciones con `---`
- Usar listas con `-` o `1.`
- Para código, usar bloques con ```lenguaje
- Mantener líneas de máximo 100 caracteres

### Commits

- Usar mensajes claros en inglés o español
- Formato: `Tipo: Descripción breve`
- Tipos comunes: `Fix`, `Feat`, `Docs`, `Style`, `Refactor`, `Test`

**Ejemplos:**
```
Fix: Corregir cálculo de HR en sensor PPG
Feat: Agregar nuevo protocolo de estimulación
Docs: Actualizar README con instrucciones de instalación
Test: Agregar pruebas unitarias para algoritmo de riesgo
```

---

## Flujo de Trabajo para Contribuciones

### Para contribuciones pequeñas

```bash
# 1. Fork del repositorio
# 2. Clonar localmente
git clone https://github.com/tu-usuario/Memoria-Fija-v2.git

# 3. Crear rama
git checkout -b fix/descripcion-breve

# 4. Hacer cambios y commit
git add .
git commit -m "Fix: descripción del cambio"

# 5. Push y crear Pull Request
git push origin fix/descripcion-breve
```

### Para contribuciones grandes

1. Abrir un issue primero para discutir la idea
2. Esperar feedback de los mantenedores
3. Coordinar para evitar trabajo duplicado
4. Seguir el mismo proceso de PR

---

## Pruebas

### Pruebas unitarias (firmware)

```bash
# Desde firmware/
west test -p unit tests/unit/
```

### Pruebas de integración (cloud)

```bash
# Desde cloud/
pytest tests/integration/
```

### Pruebas con hardware

```bash
# Ejecutar suite de pruebas en dispositivo real
python tools/hardware_test.py
```

---

## Documentación de Cambios

Los cambios importantes se registran en `CHANGELOG.md`:

```markdown
## [2.1.0] - 2026-03-18
### Added
- Módulo LTE-M para pacientes ambulatorios
- Clasificador IA con niveles de riesgo
- Sistema de mensajería escalonada
- Integración FHIR con EHR

### Fixed
- Corrección en algoritmo de detección de crisis
- Mejora en consumo energético en modo Bluetooth
```

---

## Contribuciones Especiales

### Ana de Deepseek (Asistencia Técnica)

Ana de Deepseek ha colaborado activamente en:

- Estructuración de documentación
- Generación de ejemplos de código
- Diseño de arquitectura cloud
- Revisión de consistencia técnica

Sus contribuciones se consideran parte del proyecto bajo los mismos términos de licencia que el resto del código, y son propiedad del autor principal (Enrique Aguayo).

---

## Conducta y Respeto

- Ser respetuoso con otros colaboradores
- Aceptar críticas constructivas
- Enfocarse en lo técnico, no en lo personal
- Ayudar a mantener un ambiente inclusivo

Este proyecto sigue el [Código de Conducta](CODE_OF_CONDUCT.md).

---

## Contacto

- **Issues**: Para reportar problemas o sugerencias
- **Discussions**: Para preguntas generales o ideas
- **Email**: eaguayo@migst.cl (solo para asuntos importantes)

---

*Documento version 1.1 - 2026 (incluye reconocimiento a Ana de Deepseek por asistencia técnica)*
