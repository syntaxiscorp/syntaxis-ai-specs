# Pipeline PRD → Backlog → Jira
## Flujo de Trabajo Spec-Driven

---

## 1. Visión General del Pipeline

Tres comandos agente transforman una idea de producto en tickets de Jira listos para desarrollo. Cada comando produce artefactos que el siguiente consume, asegurando trazabilidad y consistencia.

```mermaid
flowchart LR
    classDef input fill:#fff3cd,stroke:#cc9900,stroke-width:2px,color:#333300
    classDef cmd fill:#1a56db,stroke:#0f3b8c,stroke-width:2px,color:#ffffff
    classDef artifact fill:#e8d5f5,stroke:#7c3aed,stroke-width:2px,color:#2d1b4e
    classDef output fill:#bbf7d0,stroke:#15803d,stroke-width:2px,color:#14532d

    I[("Idea / Contexto del Producto")]:::input

    C1{{"/create-prd"}}:::cmd
    A1["PRD.md - Requisitos del Producto"]:::artifact

    C2{{"/create-backlog-plan"}}:::cmd
    A2["backlog.md - Epicas Priorizadas"]:::artifact
    A3["us-001..XXX.md - Historias con BDD"]:::artifact

    C3{{"/push-backlog-plan jira"}}:::cmd
    JIRA[("Jira Cloud: Epicas + Stories")]:::output

    I --> C1 --> A1 --> C2
    C2 --> A2
    C2 --> A3
    A2 --> C3
    A3 --> C3
    C3 --> JIRA

    C3 -.->|Actualiza IDs| A2
    C3 -.->|Actualiza IDs| A3
```

---

## 2. Roles y Agentes

```mermaid
flowchart LR
    classDef user fill:#e0f2fe,stroke:#0369a1,stroke-width:2px,color:#0c4a6e
    classDef agent fill:#bbf7d0,stroke:#15803d,stroke-width:2px,color:#14532d
    classDef cmd fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e3a5f

    subgraph Usuarios
        PO("Product Owner o PM"):::user
        DEV("Desarrollador"):::user
    end

    subgraph AgentesIA
        PM_BA["product-manager-ba"]:::agent
        BP["backlog-planner"]:::agent
        DA["diagram-architect"]:::agent
    end

    subgraph Comandos
        PRD_CMD["/create-prd"]:::cmd
        BACKLOG_CMD["/create-backlog-plan"]:::cmd
        DIAGRAMS_CMD["/create-diagrams"]:::cmd
        PUSH_CMD["/push-backlog-plan"]:::cmd
    end

    PO --> PRD_CMD
    PO --> BACKLOG_CMD
    PO --> DIAGRAMS_CMD
    PO --> PUSH_CMD
    DEV --> PUSH_CMD

    PRD_CMD -. Adopta .-> PM_BA
    BACKLOG_CMD -. Adopta .-> BP
    DIAGRAMS_CMD -. Adopta .-> DA
    BACKLOG_CMD -. Lee .-> PM_BA
```

---

## 3. Mapa de Artefactos y Dependencias

```mermaid
flowchart TD
    classDef artifact fill:#e8d5f5,stroke:#7c3aed,stroke-width:2px,color:#2d1b4e
    classDef ext fill:#fef3c7,stroke:#d97706,stroke-width:2px,color:#78350f

    PRD[/"PRD.md - Fuente Unica de Verdad"/]:::artifact

    subgraph Backlog
        SUM[/"backlog.md - Resumen priorizado"/]:::artifact
        US1[/"us-001.md - Historia individual"/]:::artifact
        US2[/"us-002.md - Historia individual"/]:::artifact
        USN[/"us-XXX.md - Historia individual"/]:::artifact
    end

    subgraph Diagrams
        DIAG1[/"Diagramas: Casos de Uso, Secuencia, Lean Canvas, C4, ER"/]:::artifact
    end

    JIRA[/"Jira Cloud: Epicas + Stories + Dependencias"/]:::ext

    PRD -->|create-backlog-plan| Backlog
    PRD -->|create-diagrams| Diagrams
    Diagrams -.->|Enriquecen| Backlog
    Backlog -->|push-backlog-plan| JIRA
    Diagrams -.->|Enriquecen| JIRA
    JIRA -.->|Actualiza IDs| SUM
    JIRA -.->|Actualiza IDs| US1
    JIRA -.->|Actualiza IDs| US2
    JIRA -.->|Actualiza IDs| USN
```

---

## 4. Flujo Detallado por Comando

### 4.1 `/create-prd` — De la Idea al Documento

```mermaid
flowchart LR
    classDef input fill:#fff3cd,stroke:#cc9900,stroke-width:2px,color:#333300
    classDef step fill:#f1f5f9,stroke:#64748b,stroke-width:2px,color:#1e293b
    classDef output fill:#e8d5f5,stroke:#7c3aed,stroke-width:2px,color:#2d1b4e

    IN[("Archivo de contexto")]
    IN:::input

    A[1. Analizar contexto]:::step
    B[2. Definir vision y objetivos]:::step
    C[3. Identificar usuarios]:::step
    D[4. Definir requisitos]:::step
    E[5. Disenar modelo de datos]:::step
    F[6. Definir arquitectura]:::step

    OUT[/PRD.md/]
    OUT:::output

    IN --> A --> B --> C --> D --> E --> F --> OUT
```

**Entrada**: Archivo con descripción del producto, objetivos y usuarios.
**Agente**: `product-manager-ba.md`
**Salida**: `ai-specs/specs/PRD.md`
**Idioma**: Español

**Secciones del PRD:**
- Executive Summary & Problem Statement
- Product Vision & Business Objectives
- Target Users & Personas
- Use Cases & User Stories
- Key Features & Functional Requirements
- Data Model & Entity Relationships
- System Architecture (High-Level)
- API Specification
- Non-Functional Requirements
- UX/UI Guidelines
- Risks & Assumptions
- Glossary

---

### 4.2 `/create-backlog-plan` — Del PRD al Backlog

```mermaid
flowchart LR
    classDef input fill:#fff3cd,stroke:#cc9900,stroke-width:2px,color:#333300
    classDef step fill:#f1f5f9,stroke:#64748b,stroke-width:2px,color:#1e293b
    classDef output fill:#e8d5f5,stroke:#7c3aed,stroke-width:2px,color:#2d1b4e
    classDef opt fill:#fef3c7,stroke:#d97706,stroke-width:2px,stroke-dasharray:5 5,color:#78350f

    IN1[("PRD.md")]
    IN1:::input
    IN2[("diagrams (opcional)")]
    IN2:::opt

    A[1. Descomponer en Epicas]:::step
    B[2. Generar Historias de Usuario]:::step
    C[3. Estimar Fibonacci + T-Shirt]:::step
    D[4. Priorizar RICE + MoSCoW]:::step
    E[5. Escribir criterios BDD]:::step
    F[6. Definir plan de liberacion]:::step

    OUT1[/backlog.md/]
    OUT1:::output
    OUT2[/us-XXX.md/]
    OUT2:::output

    IN1 --> A
    IN2 -.-> A
    A --> B --> C --> D --> E --> F
    F --> OUT1
    F --> OUT2
```

**Entrada**: `PRD.md` (obligatorio) + `diagrams/*.md` (opcional)
**Agente**: `backlog-planner.md`
**Salida**: `backlog/backlog.md` + `backlog/us-XXX.md`

**Cada Historia de Usuario incluye:**

```mermaid
flowchart TD
    classDef req fill:#f1f5f9,stroke:#64748b,stroke-width:2px,color:#1e293b
    classDef bdd fill:#dcfce7,stroke:#15803d,stroke-width:2px,color:#14532d
    classDef tech fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#1e3a5f

    A["Historia de Usuario"]:::req
    B["Formato: Como... quiero... para..."]:::req
    C["Estimacion: Fibonacci + T-Shirt"]:::req
    D["Prioridad: RICE + MoSCoW"]:::req
    E["Criterios BDD: Gherkin 3+ escenarios"]:::bdd
    F["Notas Tecnicas: archivos, endpoints"]:::tech
    G["Dependencias: bloqueado por / bloquea"]:::tech
    H["Definition of Done: tests, code review"]:::req

    A --> B --> C --> D --> E --> F --> G --> H
```

---

## 5. Metodologias de Estimacion y Priorizacion

Cada Historia de Usuario se estima y prioriza usando tres metodologias complementarias que trabajan en conjunto:

```mermaid
flowchart LR
    classDef fib fill:#fef3c7,stroke:#d97706,stroke-width:2px,color:#78350f
    classDef rice fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#1e3a5f
    classDef moscow fill:#dcfce7,stroke:#15803d,stroke-width:2px,color:#14532d
    classDef output fill:#e8d5f5,stroke:#7c3aed,stroke-width:2px,color:#2d1b4e

    subgraph Estimacion
        FIB["Fibonacci: Cuanto esfuerzo requiere?"]:::fib
    end

    subgraph Priorizacion
        RICE["RICE: Que tan urgente es?"]:::rice
        MSC["MoSCoW: Que tan critico es?"]:::moscow
    end

    RESULT["Cada US obtiene: Story Points + RICE Score + MoSCoW Categoria"]:::output

    FIB --> RESULT
    RICE --> RESULT
    MSC --> RESULT
```

### 5.1 Fibonacci — Estimacion de Esfuerzo

La secuencia de Fibonacci (1, 2, 3, 5, 8, 13, 21) se usa para asignar Story Points a cada historia. La secuencia fuerza al equipo a tomar decisiones: la diferencia entre 1 y 2 es pequena, pero entre 13 y 21 es grande. Esto evita analisis de precision falsa.

```mermaid
flowchart LR
    classDef small fill:#dcfce7,stroke:#15803d,stroke-width:2px,color:#14532d
    classDef medium fill:#fef3c7,stroke:#d97706,stroke-width:2px,color:#78350f
    classDef large fill:#fecaca,stroke:#dc2626,stroke-width:2px,color:#7f1d1d
    classDef xlarge fill:#e8d5f5,stroke:#7c3aed,stroke-width:2px,color:#2d1b4e

    1["1 punto - Tarea trivial"]:::small
    2["2 puntos - Tarea simple"]:::small
    3["3 puntos - Tarea media"]:::medium
    5["5 puntos - Historia mediana"]:::medium
    8["8 puntos - Historia compleja"]:::large
    13["13 puntos - Historia grande"]:::large
    21["21 puntos - Epica (debe dividirse)"]:::xlarge

    1 --> 2 --> 3 --> 5 --> 8 --> 13 --> 21
```

**Reglas de la estimacion Fibonacci:**

| Puntos | Significado | Ejemplo | Accion si se asigna |
|---|---|---|---|
| 1 | Ajuste trivial: cambiar texto, color | Cambiar label de un boton | Adelante |
| 2 | Tarea simple: un archivo, sin riesgo | Agregar campo a un formulario existente | Adelante |
| 3 | Tarea media: 2-3 archivos, riesgo bajo | Nuevo endpoint GET simple | Adelante |
| 5 | Historia mediana: nuevo componente o servicio | CRUD completo de una entidad pequena | Adelante |
| 8 | Historia compleja: multiples componentes | Filtro recursivo en arbol jerarquico | Considerar dividir |
| 13 | Historia muy grande: riesgo alto | Integracion con API externa compleja | Debe dividirse en 2+ historias |
| 21 | Demasiado grande: epica, no historia | Modulo completo de autenticacion | Dividir obligatoriamente |

**Analogia para la demo:** Asi como en la naturaleza las caracolas siguen la secuencia Fibonacci para crecer, nosotros la usamos para que el equipo no finja tener una precision que no tiene. No sabemos si una historia es "exactamente 4" o "exactamente 6", pero si sabemos si es "mas como un 3" o "mas como un 8". La brecha entre numeros se agranda intencionalmente para aceptar la incertidumbre.

### 5.2 RICE — Priorizacion por Impacto

RICE es un sistema de scoring que asigna un numero a cada historia. Mientras mas alto el score, mas prioritaria es la historia. Se calcula como:

```
RICE Score = (Reach x Impact x Confidence) / Effort
```

```mermaid
flowchart TD
    classDef metric fill:#f8fafc,stroke:#475569,stroke-width:2px,color:#1e293b
    classDef formula fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#1e3a5f
    classDef result fill:#bbf7d0,stroke:#15803d,stroke-width:2px,color:#14532d

    R[("Reach: Cuantos usuarios por mes?")]:::metric
    I[("Impact: Que tanto impacto?")]:::metric
    C[("Confidence: Que tan seguros?")]:::metric
    E[("Effort: Story Points")]:::metric

    MULT[("(R x I x C) / E")]:::formula
    SCORE["RICE Score"]:::result

    R --> MULT
    I --> MULT
    C --> MULT
    E --> MULT
    MULT --> SCORE
```

**Desglose de cada componente:**

| Componente | Que mide | Escala | Como se asigna |
|---|---|---|---|
| **Reach** | Cuantos usuarios impacta en un periodo | Numero de usuarios/mes | 1 = muy pocos, 10 = algunos, 100 = muchos, 1000 = todos |
| **Impact** | Que tan profundo es el impacto | 0.25x a 3x | 0.25 = minimo, 0.5 = bajo, 1 = medio, 2 = alto, 3 = masivo |
| **Confidence** | Que tan seguros estamos de las cifras | Porcentaje (20% a 100%) | 20% = suposicion, 50% = estimacion con datos parciales, 80% = datos solidos, 100% = dato comprobado |
| **Effort** | Cuanto esfuerzo requiere | Story Points (Fibonacci) | Mismo valor de la estimacion Fibonacci |

**Ejemplo practico:**

| Historia | Reach | Impact | Confidence | Effort | RICE Score | Orden |
|---|---|---|---|---|---|---|
| Filtro por columna | 500 | 2x | 80% | 5 | (500x2x0.8)/5 = 160 | 1ro |
| Badge de restricciones | 300 | 1x | 50% | 3 | (300x1x0.5)/3 = 50 | 2do |
| Expandir todo | 100 | 0.5x | 50% | 1 | (100x0.5x0.5)/1 = 25 | 3ro |
| Carga masiva Excel | 50 | 3x | 20% | 13 | (50x3x0.2)/13 = 2.3 | 4to |

### 5.3 MoSCoW — Categorizacion de Prioridad

MoSCoW clasifica cada historia en cuatro categorias. Se usa como filtro final despues de RICE para definir que entra en cada fase de liberacion.

```mermaid
flowchart TD
    classDef must fill:#dc2626,stroke:#991b1b,stroke-width:2px,color:#ffffff
    classDef should fill:#d97706,stroke:#92400e,stroke-width:2px,color:#ffffff
    classDef could fill:#2563eb,stroke:#1e3a8a,stroke-width:2px,color:#ffffff
    classDef wont fill:#6b7280,stroke:#374151,stroke-width:2px,color:#ffffff

    MUST["MUST HAVE: Imprescindible para MVP"]:::must
    SHOULD["SHOULD HAVE: Importante pero no critico"]:::should
    COULD["COULD HAVE: Deseable si hay tiempo"]:::could
    WONT["WONT HAVE: Explicitamente fuera de alcance"]:::wont
```

| Categoria | Significado | Sin esto... | Que entra | Mapeo a Jira |
|---|---|---|---|---|
| **Must Have** | Esencial para el lanzamiento. Sin esto el producto no tiene sentido. | El producto falla o no se puede lanzar | Funcionalidad core, flujos criticos, bugs bloqueantes | Highest Priority |
| **Should Have** | Importante pero se puede lanzar sin esto. Se incluye si el tiempo lo permite. | El producto funciona pero pierde valor significativo | Mejoras importantes, features de retention | High Priority |
| **Could Have** | Deseable pero no necesario. Se incluye solo si sobra tiempo. | El producto funciona igual, solo pierde "cosas lindas" | Mejoras cosmeticas, features nice-to-have | Medium Priority |
| **Won't Have** | Explicitamente acordado que no se hara en esta version. | Nada, nunca se prometio | Features para futuras versiones, ideas descartadas | No se crea en Jira |

**Regla 60/20/20 para MVP:**
- **60%** Must Have (funcionalidad core del MVP)
- **20%** Should Have (mejoras importantes post-MVP inmediato)
- **20%** Could Have (deseables para versiones futuras)
- Won't Have queda fuera del calculo

### 5.4 Las Tres Metodologias en Conjunto

Las tres metodologias se aplican en secuencia durante `/create-backlog-plan`:

```mermaid
flowchart LR
    classDef step fill:#f1f5f9,stroke:#64748b,stroke-width:2px,color:#1e293b
    classDef fib fill:#fef3c7,stroke:#d97706,stroke-width:2px,color:#78350f
    classDef rice fill:#dbeafe,stroke:#1d4ed8,stroke-width:2px,color:#1e3a5f
    classDef msc fill:#dcfce7,stroke:#15803d,stroke-width:2px,color:#14532d

    ESTIMATE[("Historia de Usuario")]

    F["Paso 1: Fibonacci
    Se asigna Story Points
    1, 2, 3, 5, 8, 13, 21"]:::fib

    R["Paso 2: RICE
    Se calcula score
    (Reach x Impact x Confidence) / Effort"]:::rice

    M["Paso 3: MoSCoW
    Se asigna categoria
    Must / Should / Could / Wont"]:::msc

    RESULT["Historia lista con:
    Story Points + RICE Score + MoSCoW"]:::step

    ESTIMATE --> F --> R --> M --> RESULT
```

**Ejemplo completo de una historia priorizada:**

```
US-001: Filtrar arbol por columna

  Fibonacci: 5 puntos (historia mediana: nuevo algoritmo recursivo)
  RICE:      (500 x 2 x 0.8) / 5 = 160 (score mas alto del backlog)
  MoSCoW:    Must Have (sin filtros el arbol es inusable)
  Posicion:  #1 en el backlog priorizado
  Fase:      MVP
```

### 5.5 Mapeo a Jira

Cuando se ejecuta `/push-backlog-plan jira`, los valores se mapean directamente:

| Metodologia | Valor Local | Destino en Jira |
|---|---|---|
| Fibonacci | Story Points (ej: 5) | Campo `Story Points` en la Story |
| RICE | Score (ej: 160) | Se incluye en la descripcion ADF como referencia |
| MoSCoW | Must / Should / Could | Se mapea a Priority: Highest / High / Medium |

---

### 4.3 `/push-backlog-plan jira` — Del Backlog a Jira

```mermaid
flowchart LR
    classDef input fill:#fff3cd,stroke:#cc9900,stroke-width:2px,color:#333300
    classDef step fill:#f1f5f9,stroke:#64748b,stroke-width:2px,color:#1e293b
    classDef output fill:#bbf7d0,stroke:#15803d,stroke-width:2px,color:#14532d

    IN1[("backlog/backlog.md")]:::input
    IN2[("backlog/us-XXX.md")]:::input
    IN3[("diagrams (opcional)")]:::input

    V[1. Validar consistencia]:::step
    E[2. Crear Epicas en Jira]:::step
    S[3. Crear Stories en Jira]:::step
    D[4. Enlazar dependencias]:::step
    L[5. Agregar labels mvp/v1.1]:::step
    U[6. Actualizar artefactos con IDs]:::step

    EPIC["Epicas en Jira"]:::output
    STORY["Stories en Jira con ADF"]:::output
    DEP["Dependencias bloqueado por"]:::output

    IN1 --> V
    IN2 --> V
    IN3 -.-> V
    V --> E --> S --> D --> L --> U
    E --> EPIC
    S --> STORY
    D --> DEP
```

**Entrada**: `backlog/backlog.md` + `backlog/us-XXX.md` + `diagrams/*.md` (opcional)
**Salida**: Issues en Jira (Épicas + Stories con dependencias)
**Post-ejecución**: `backlog.md` y `us-XXX.md` se actualizan con los IDs de Jira

**Formato de descripción en Jira (ADF):**

| Campo | Valor | Notas |
|---|---|---|
| Issue Type | `Epic` para épicas, `Story` para US | Mapeo 1:1 |
| Summary | Título de la épica o historia | De `us-XXX.md` |
| Description | ADF JSON (nunca markdown plano) | Evita doble escape de `\n` |
| Story Points | Valor Fibonacci | De la estimación |
| Priority | Highest/High/Medium | Mapeo de MoSCoW |
| Epic Link | ID de la épica padre | Creada primero |
| Labels | `mvp`, `v1.1`, `v1.2` | Según plan de liberación |

---

## 6. Timeline del Pipeline

```mermaid
gantt
    title Ciclo de Vida: Idea a Jira
    dateFormat  YYYY-MM-DD
    axisFormat  %d

    section Fase 1: Estrategia
    /create-prd :done, f1, 2026-06-01, 1d
    PRD.md generado :done, after f1, 1d

    section Fase 2: Planeacion
    /create-backlog-plan :done, f2, after f1, 1d
    backlog.md + us-XXX.md :done, after f2, 1d

    section Fase 2b: Diagramas (opcional)
    /create-diagrams :done, f2b, after f1, 1d
    diagrams/*.md :done, after f2b, 1d

    section Fase 3: Publicacion
    /push-backlog-plan jira :active, f3, after f2, 1d
    Epicas + Stories en Jira :active, after f3, 1d
```

---

## 7. Reglas de Consistencia

| Componente | Regla |
|---|---|
| **Épicas** | Nombres y descripciones deben coincidir entre `backlog.md` y Jira |
| **Historias** | Cada `us-XXX.md` se mapea 1:1 con una Story en Jira |
| **Story Points** | El valor en el archivo local debe coincidir con Jira |
| **Prioridad** | MoSCoW a Jira Priority: Must=Highest, Should=High, Could=Medium |
| **Dependencias** | Relaciones "bloqueado por" en Jira reflejan el mapa en `backlog.md` |
| **Labels** | `mvp`, `v1.1`, `v1.2` según plan de liberación |
| **Post-sync** | `backlog.md` y `us-XXX.md` se actualizan automáticamente con IDs de Jira |

---

## 8. Beneficios del Flujo Spec-Driven

### 8.1 Tiempos

```mermaid
flowchart LR
    classDef before fill:#fecaca,stroke:#dc2626,stroke-width:2px,color:#7f1d1d
    classDef after fill:#bbf7d0,stroke:#15803d,stroke-width:2px,color:#14532d

    subgraph Antes["Sin el flujo (manual)"]
        A1["PM escribe PRD en Word: 2-3 semanas"]:::before
        A2["Reuniones de refinamiento: 4-8 horas por sprint"]:::before
        A3["PM escribe cada historia a mano: 30 min cada una"]:::before
        A4["Bug por malentendido: 1-3 dias de retrabajo"]:::before
    end

    subgraph Despues["Con el flujo (spec-driven)"]
        D1["PRD generado por IA: 1 ejecucion"]:::after
        D2["Historias con BDD y estimacion: 1 ejecucion"]:::after
        D3["Sync a Jira automatico: 1 ejecucion"]:::after
        D4["Cero ambiguedad: misma informacion para todos"]:::after
    end
```

| Actividad | Sin el flujo | Con el flujo | Ahorro |
|---|---|---|---|
| Crear PRD | 2-3 semanas de ida y vuelta | 1 ejecucion del comando | ~15 dias |
| Refinar backlog para un sprint | 4-8 horas de reunion | 1 ejecucion del comando | ~6 horas |
| Escribir una historia de usuario | 30 min manual + correcciones | 0 min (se genera automaticamente) | 30 min por historia |
| Sincronizar backlog a Jira | 2-3 horas copiando y pegando | 1 ejecucion del comando | ~2.5 horas |
| Resolver ambiguedades en desarrollo | 1-3 dias de retrabajo | 0 (BDD elimina ambiguedad) | 1-3 dias |
| Total estimado por sprint de 2 semanas | 3-5 dias administrativos | 3 ejecuciones de comando (~10 min) | ~85% menos tiempo administrativo |

**Impacto en el equipo:** El PM recupera ~15 horas semanales que puede invertir en estrategia, investigacion de usuarios y alineacion con stakeholders. El desarrollador nunca se bloquea esperando aclaraciones.

### 8.2 Personal

```mermaid
flowchart TD
    classDef problem fill:#fecaca,stroke:#dc2626,stroke-width:2px,color:#7f1d1d
    classDef solution fill:#bbf7d0,stroke:#15803d,stroke-width:2px,color:#14532d

    P1["Problema: PM es cuello de botella"]:::problem
    P2["Problema: Desarrolladores bloqueados esperando respuestas"]:::problem
    P3["Problema: Curva de aprendizaje para nuevos miembros"]:::problem
    P4["Problema: Conocimiento tacito solo en la cabeza del PM"]:::problem

    S1["Solucion: IA aumenta al PM, no lo reemplaza"]:::solution
    S2["Solucion: Historias autocontenidas, el dev no necesita preguntar"]:::solution
    S3["Solucion: Documentacion viva siempre actualizada"]:::solution
    S4["Solucion: Todo queda escrito en PRD + backlog + diagramas"]:::solution

    P1 --> S1
    P2 --> S2
    P3 --> S3
    P4 --> S4
```

**Como impacta a cada rol:**

| Rol | Sin el flujo | Con el flujo |
|---|---|---|
| **Product Manager** | Escribe documentos manualmente, asiste a reuniones de refinamiento, responde preguntas uno a uno | Define la vision, ejecuta comandos, valida resultados, se enfoca en estrategia |
| **Desarrollador Backend** | Recibe historias Vagas, espera aclaraciones del PM, asume comportamientos | Recibe historias con BDD, endpoints definidos, plan DDD paso a paso |
| **Desarrollador Frontend** | Adivina diseno, corrige malentendidos en code review | Recibe criterios BDD, plan de componentes, routing definido |
| **Nuevo Integrante** | Lee documentacion dispersa, pregunta a companeros, comete errores | Lee PRD + backlog + diagramas + planes, todo coherente y actualizado |
| **QA / Tester** | Crea casos basados en suposicion | Usa escenarios Gherkin ya escritos en las historias |

### 8.3 Reiteracion (Iteraciones y Consistencia)

El flujo produce el mismo nivel de calidad en cada iteracion. No hay "sprints buenos" y "sprints malos" dependiendo de que tan cansado este el PM.

```mermaid
flowchart LR
    classDef sprint fill:#e8d5f5,stroke:#7c3aed,stroke-width:2px,color:#2d1b4e
    classDef quality fill:#bbf7d0,stroke:#15803d,stroke-width:2px,color:#14532d

    S1["Sprint 1: Misma calidad"]:::sprint
    S2["Sprint 2: Misma calidad"]:::sprint
    S3["Sprint 3: Misma calidad"]:::sprint
    SN["Sprint N: Misma calidad"]:::sprint

    Q["Calidad consistente en cada historia:
    BDD + Estimacion + Priorizacion + Notas Tecnicas"]:::quality

    S1 --> Q
    S2 --> Q
    S3 --> Q
    SN --> Q
```

| Aspecto | Sin el flujo (cada sprint es diferente) | Con el flujo (cada sprint es consistente) |
|---|---|---|
| Formato de historias | Cambia segun quien las escribe y su humor | Identico siempre: BDD, estimacion, prioridad, notas tecnicas |
| Profundidad del detalle | Depende del tiempo disponible del PM | Siempre completa: 3+ escenarios Gherkin, dependencias, DoD |
| Criterios de aceptacion | "El usuario deberia poder..." (ambiguo) | Given / When / Then (ejecutable por QA) |
| Estimacion | Adivinanza sin contexto | Fibonacci + RICE con rationale documentado |
| Documentacion | Se desactualiza y nadie la actualiza | Se actualiza automaticamente en cada implementacion |
| Curva de aprendizaje | Cada sprint el equipo aprende de nuevo el proceso | El proceso es siempre el mismo, predecible |

### 8.4 Otros Puntos Clave

| Beneficio | Descripcion | Impacto |
|---|---|---|
| **Trazabilidad Total** | Cada linea de codigo se rastrea hasta un requisito del PRD pasando por una historia de usuario. | Auditoria, compliance, onboarding |
| **Fuente Unica de Verdad** | El PRD es el documento maestro. Cualquier cambio en requisitos se refleja automaticamente en backlog y diagramas. | Equipo siempre alineado |
| **Reduccion de Riesgos** | Validacion en cada paso: el PRD se valida antes del backlog, el backlog antes de Jira, el plan antes del codigo. | Errores tempranos cuestan menos |
| **Calidad Incorporada** | 90% de cobertura de tests, BDD escrito antes del codigo, DoD explicito en cada historia. | Menos bugs en produccion |
| **Paralelismo Backend + Frontend** | Planes separados para BE y FE permiten que dos developers trabajen el mismo ticket simultaneamente. | Hasta 50% mas rapido por ticket |
| **Onboarding Acelerado** | Un nuevo desarrollador puede leer PRD + backlog + diagramas + planes y entender el producto completo en horas, no semanas. | Nuevos miembros productivos el dia 1 |
| **Conocimiento Persistente** | Todo queda documentado: decisiones, estimaciones, dependencias. No hay conocimiento tacito que se vaya cuando alguien renuncia. | El equipo es resiliente a rotacion |
| **Estandarizacion** | Todos los proyectos siguen el mismo proceso. Un dev puede moverse entre proyectos sin reaprender el flujo. | Escalabilidad del equipo |

### 8.5 Antes vs. Despues: El Cambio Radical

```mermaid
flowchart TD
    classDef bad fill:#fecaca,stroke:#dc2626,stroke-width:2px,color:#7f1d1d
    classDef good fill:#bbf7d0,stroke:#15803d,stroke-width:2px,color:#14532d

    subgraph Antes2["Flujo Tradicional (sin comandos)"]
        B1["PM escribe spec en Word"]:::bad
        B2["PM envia por email al equipo"]:::bad
        B3["Dev lee e interpreta"]:::bad
        B4["Dev pregunta: 'Que pasa con X?'"]:::bad
        B5["PM responde 2 dias despues"]:::bad
        B6["Dev implementa lo que entendio"]:::bad
        B7["QA rechaza: no coincide con lo pedido"]:::bad
        B8["Retrabajo: 2-3 dias perdidos"]:::bad
    end

    subgraph Despues2["Flujo Spec-Driven (con comandos)"]
        G1["PM ejecuta /create-prd"]:::good
        G2["PRD generado automaticamente"]:::good
        G3["PM ejecuta /create-backlog-plan"]:::good
        G4["Backlog con BDD + estimacion + prioridad"]:::good
        G5["PM ejecuta /push-backlog-plan jira"]:::good
        G6["Epicas y Stories creadas en Jira"]:::good
        G7["Dev implementa con plan detallado"]:::good
        G8["QA valida contra BDD: pasa o no pasa"]:::good
    end

    B1 --> B2 --> B3 --> B4 --> B5 --> B6 --> B7 --> B8
    G1 --> G2 --> G3 --> G4 --> G5 --> G6 --> G7 --> G8
```

**Impacto en numeros (estimado para un equipo de 5 personas en un sprint de 2 semanas):**

| Metrica | Flujo Tradicional | Flujo Spec-Driven | Mejora |
|---|---|---|---|
| Tiempo administrativo del PM | 3-5 dias por sprint | 30 minutos por sprint | **85% menos** |
| Historias sin ambiguedad | ~40% | ~100% | **+60%** |
| Retrabajo por malentendidos | 2-3 dias por sprint | Casi cero | **~95% menos** |
| Tiempo de onboarding nuevo dev | 2-4 semanas | 2-4 dias | **~75% mas rapido** |
| Historias por sprint | 5-8 | 8-12 | **+50%** |
| Documentacion actualizada | Casi nunca | Siempre | **100% del tiempo** |

---

## 9. Resumen para la Demo

```mermaid
flowchart LR
    classDef phase fill:#eff6ff,stroke:#2563eb,stroke-width:3px,color:#1e3a5f
    classDef cmd fill:#1a56db,stroke:#0f3b8c,stroke-width:2px,color:#ffffff
    classDef artifact fill:#e8d5f5,stroke:#7c3aed,stroke-width:2px,color:#2d1b4e
    classDef result fill:#bbf7d0,stroke:#15803d,stroke-width:2px,color:#14532d

    P1["Fase 1: Estrategia"]:::phase
    P2["Fase 2: Planeacion"]:::phase
    P3["Fase 3: Publicacion"]:::phase

    C1("/create-prd"):::cmd
    C2("/create-backlog-plan"):::cmd
    C3("/push-backlog-plan"):::cmd

    A1["PRD.md"]:::artifact
    A2["backlog.md + us-XXX.md"]:::artifact
    A3["Jira: Epicas + Stories"]:::result

    P1 --> C1 --> A1
    P2 --> C2 --> A2
    P3 --> C3 --> A3
```

| Fase | Comando | Qué Produce | Tiempo Estimado |
|---|---|---|---|
| **1. Estrategia** | `/create-prd` | `PRD.md` — Documento de requisitos | 1 ejecución |
| **2. Planeación** | `/create-backlog-plan` | `backlog.md` + `us-XXX.md` — Backlog priorizado con BDD | 1 ejecución |
| **2b. (Opcional)** | `/create-diagrams` | `diagrams/*.md` — 5 diagramas visuales | 1 ejecución |
| **3. Publicación** | `/push-backlog-plan jira` | Épicas y Stories en Jira con dependencias | 1 ejecución |
