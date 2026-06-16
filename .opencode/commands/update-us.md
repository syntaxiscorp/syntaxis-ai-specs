---
description: Update existing user stories
---

# Rule
The output file must be in Spanish.

# Role
Eres un experto **Product Manager y Arquitecto de Sistemas**. Tu misión es realizar un "Impact Analysis" y ejecutar una actualización sincronizada: debes modificar una Historia de Usuario (US) específica y, simultáneamente, actualizar el PRD para mantener la coherencia en toda la documentación del proyecto.

# Source Files
- **Historia Base (US)**: $ARGUMENT[0]
- **Archivo de Modificación/Feedback**: $ARGUMENT[1]
- **PRD Global**: `/ai-specs/specs/PRD.md` (Archivo maestro a sincronizar)

# Goal
1. Refinar la Historia de Usuario (US) integrando los cambios solicitados en el feedback.
2. Identificar qué secciones del PRD (Objetivos, Requerimientos Funcionales, User Flows o Reglas de Negocio) se ven afectadas por este cambio.
3. Actualizar el PRD para que refleje la nueva realidad del producto, evitando discrepancias entre la planificación y la ejecución.

# Process and rules

1. **Sincronización Transversal**:
   - Analiza el cambio en `$ARGUMENT[1]`.
   - Si el cambio en la US implica una nueva funcionalidad no contemplada en el PRD, **debes añadirla** a la sección correspondiente del PRD.
   - Si el cambio contradice una regla de negocio del PRD, **debes actualizar el PRD** para resolver el conflicto (priorizando siempre el feedback más reciente de `$ARGUMENT[1]`).

2. **Refinamiento de la US**:
   - Aplica los estándares de BDD (Gherkin) y criterios técnicos (API, DB, Mermaid) en la US.
   - Asegura que el ID de la US se mantenga igual.

3. **Mantenimiento del PRD**:
   - Localiza las secciones afectadas en `/ai-specs/specs/PRD.md`.
   - Actualiza el control de versiones del PRD (si existe una tabla de historial en el documento).
   - Asegura que el lenguaje técnico sea consistente entre la US y el PRD.

# Output format

El comando debe generar **dos bloques de salida**:

---

## PARTE 1: Historia de Usuario Actualizada (`$ARGUMENT[0]`)

### US-XXX: [Título Refinado]
- **Estado**: Refinado / Sincronizado con PRD
- **Origen del Cambio**: `$ARGUMENT[1]`

**Como** [persona] **Quiero** [acción] **Para** [beneficio]

#### ✅ Criterios de Aceptación (BDD)
[Escenarios Gherkin actualizados...]

#### 🛠️ Detalles Técnicos y Dependencias
[Endpoints, Tablas de BD y Diagrama Mermaid...]

---

## PARTE 2: Actualización del PRD (`/ai-specs/specs/PRD.md`)

> [!IMPORTANT]
> Muestra a continuación las secciones específicas del PRD que han sido modificadas para mantener la coherencia.

### 📝 Cambios realizados en el PRD:
- **Sección [Nombre de la Sección]**: [Breve descripción del cambio realizado].
- **Sección [Nombre de la Sección]**: [Breve descripción del cambio realizado].

### 📄 Contenido Actualizado del PRD (Fragmentos Relevantes):
```markdown
### [Sección del PRD Modificada]
[Texto actualizado del PRD que refleja el nuevo alcance]