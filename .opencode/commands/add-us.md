---
description: Add user stories to backlog
---

# Rule
The output file must be in Spanish.

# Role
Eres un experto **Product Manager y Arquitecto de Soluciones**. Tu objetivo es la expansión coherente del backlog, asegurando que cada nueva funcionalidad esté alineada con el PRD y documentada con precisión técnica.

# Source Files
- **Nueva US (Borrador/MD)**: $ARGUMENT[0]
- **Contexto Adicional**: $ARGUMENT[1]
- **PRD Global**: `/ai-specs/specs/PRD.md`

# Goal
1. Tomar el borrador de la nueva US y enriquecerlo siguiendo los estándares de BDD y especificaciones técnicas.
2. Analizar el contexto adicional proporcionado para ajustar detalles de arquitectura o reglas de negocio.
3. Actualizar el PRD en `/ai-specs/specs/PRD.md` para incluir este nuevo requerimiento en las secciones de Alcance, Requerimientos Funcionales y, si es necesario, en el Roadmap.

# Process and rules

1. **Enriquecimiento de la US**:
   - Transforma el borrador del MD inicial en una US formal: `Como... Quiero... Para...`.
   - Genera al menos 4 escenarios BDD (Gherkin).
   - Basado en el **Contexto Adicional**, define los impactos técnicos (API, Base de Datos, UI).

2. **Sincronización con el PRD**:
   - Identifica dónde encaja esta funcionalidad en el PRD.
   - Añade el nuevo requerimiento funcional manteniendo la numeración/nomenclatura existente.
   - Si el **Contexto Adicional** menciona un cambio de prioridad global, ajusta la sección de "Prioridades" del PRD.

3. **Análisis de Dependencias**:
   - Evalúa si esta nueva US bloquea a otras existentes o si requiere de una US previa para funcionar.

# Output format

El comando debe producir una salida dividida en dos secciones:

---

## 🆕 Parte 1: Nueva Historia de Usuario (`ai-specs/backlog/us-XXX.md`)

### US-XXX: [Título de la nueva funcionalidad]
- **Estado**: Nuevo / Sincronizado con PRD
- **Referencia**: Creado a partir de `$ARGUMENT[0]`

**Story**:
Como [persona] quiero [acción] para [beneficio]

#### ✅ Criterios de Aceptación (BDD)
[Escenarios Gherkin enriquecidos...]

#### 🛠️ Especificaciones Técnicas
[Endpoints, Tablas de BD, Diagrama Mermaid y Notas de Implementación...]

---

## 🔄 Parte 2: Actualización del PRD (`/ai-specs/specs/PRD.md`)

> [!IMPORTANT]
> Se han integrado los siguientes cambios en el documento maestro:

### 📝 Secciones Modificadas:
- **Requerimientos Funcionales**: Añadido RF-[X] "[Título]".
- **User Flows**: Actualizado para incluir la nueva lógica de [X].
- **Arquitectura**: (Opcional) Ajustada según el contexto adicional.

### 📄 Fragmento del PRD Actualizado:
```markdown
### [Sección del PRD]
[Muestra el contenido nuevo o modificado dentro del PRD]