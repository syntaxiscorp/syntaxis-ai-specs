# Diagrama de Flujo: Pipeline Spec-Driven de Comandos Agente
## Flujo de Trabajo Completo

---

## 1. Vision General del Pipeline

El pipeline completo abarca 17 comandos agente organizados en 7 fases. Transforma una idea de producto en bruto en codigo funcionando y tickets en Jira/GitHub, pasando por PRD, backlog, diagramas, planes de implementacion y control de versiones.

```mermaid
flowchart LR
    classDef strategic fill:#e1d5e7,stroke:#9673a6,stroke-width:2px,color:#1e293b
    classDef planning fill:#dae8fc,stroke:#6c8ebf,stroke-width:2px,color:#1e293b
    classDef execution fill:#d5e8d4,stroke:#82b366,stroke-width:2px,color:#1e293b
    classDef output fill:#ffe6cc,stroke:#d79b00,stroke-width:2px,color:#1e293b

    P1["Fase 1: Estrategia"]:::strategic
    P2["Fase 2-3: Planeacion"]:::planning
    P3["Fase 4-7: Ejecucion"]:::execution
    OUT["Code + Jira"]:::output

    P1 --> P2 --> P3 --> OUT
```

**Las 7 fases del pipeline:**
1. Estrategia de Producto (`/create-prd`)
2. Backlog y Diagramas (`/create-backlog-plan`, `/create-diagrams`)
3. Mantenimiento del Backlog (`/add-us`, `/update-us`, `/delete-us`)
4. Planificacion de Tickets (`/plan-backend-ticket`, `/plan-frontend-ticket`)
5. Implementacion (`/develop-backend`, `/develop-frontend`)
6. Commit y PR (`/commit`)
7. Sincronizacion con Plataforma (`/push-backlog-plan`)

---

## 2. Actores, Comandos y Artefactos

### 2.1 17 Comandos y sus Roles

```mermaid
flowchart TD
    classDef cmd fill:#2563eb,stroke:#1d4ed8,stroke-width:2px,color:#fff
    classDef phase fill:#f0f9ff,stroke:#0284c7,stroke-width:3px,color:#1e293b

    subgraph Fase1["Fase 1: Estrategia"]
        C1["/create-prd"]:::cmd
    end

    subgraph Fase2["Fase 2: Backlog + Diagramas"]
        C2["/create-backlog-plan"]:::cmd
        C3["/create-diagrams"]:::cmd
    end

    subgraph Fase3["Fase 3: Mantenimiento"]
        C4["/add-us"]:::cmd
        C5["/update-us"]:::cmd
        C6["/delete-us"]:::cmd
        C7["/enrich-us"]:::cmd
    end

    subgraph Fase4["Fase 4: Planificacion"]
        C8["/plan-backend-ticket"]:::cmd
        C9["/plan-frontend-ticket"]:::cmd
    end

    subgraph Fase5["Fase 5: Implementacion"]
        C10["/develop-backend"]:::cmd
        C11["/develop-frontend"]:::cmd
    end

    subgraph Fase6["Fase 6: Versionado"]
        C12["/commit"]:::cmd
    end

    subgraph Fase7["Fase 7: Sincronizacion"]
        C13["/push-backlog-plan"]:::cmd
    end

    subgraph Aux["Auxiliares"]
        C14["/update-docs"]:::cmd
        C15["/explain"]:::cmd
        C16["/meta-prompt"]:::cmd
    end

    Fase1 --> Fase2 --> Fase3 --> Fase4 --> Fase5 --> Fase6 --> Fase7
```

### 2.2 Mapa de Artefactos

| Artefacto | Producido Por | Consumido Por | Descripcion |
|---|---|---|---|
| `ai-specs/specs/PRD.md` | `/create-prd` | `create-backlog-plan`, `create-diagrams`, `add-us`, `update-us`, `delete-us` | Requisitos del producto |
| `ai-specs/backlog/backlog.md` | `/create-backlog-plan` | `push-backlog-plan` | Epicas priorizadas |
| `ai-specs/backlog/us-XXX.md` | `/create-backlog-plan`, `/add-us` | `enrich-us`, `update-us`, `delete-us`, `plan-backend-ticket`, `plan-frontend-ticket` | Historias con BDD |
| `ai-specs/diagrams/*.md` | `/create-diagrams` | `create-backlog-plan`, `push-backlog-plan` | Diagramas visuales |
| `docs/plans/[ID]_backend.md` | `/plan-backend-ticket` | `/develop-backend` | Plan backend DDD |
| `docs/plans/[ID]_frontend.md` | `/plan-frontend-ticket` | `/develop-frontend` | Plan frontend React |
| Git Commit + PR | `/commit` | Code review + merge | Codigo versionado |
| Jira / GitHub Issues | `/push-backlog-plan` | Sprint planning | Items trackeables |

---

## 3. Flujo Completo del Pipeline

```mermaid
flowchart TD
    classDef agent fill:#d5e8d4,stroke:#82b366,stroke-width:2px,color:#1e293b
    classDef cmd fill:#dae8fc,stroke:#6c8ebf,stroke-width:2px,color:#1e293b
    classDef artifact fill:#e1d5e7,stroke:#9673a6,stroke-width:2px,color:#1e293b
    classDef external fill:#f5f5f5,stroke:#666,stroke-width:2px,stroke-dasharray:5 5,color:#1e293b

    ProjectContext["Contexto del Proyecto"]

    subgraph Estrategia
        C1["/create-prd"]:::cmd
        PM_BA["product-manager-ba"]:::agent
        PRD["PRD.md"]:::artifact
    end

    subgraph Planeacion
        C2["/create-backlog-plan"]:::cmd
        C2b["/create-diagrams"]:::cmd
        BP["backlog-planner"]:::agent
        DA["diagram-architect"]:::agent
        BACKLOG["backlog.md"]:::artifact
        US["us-XXX.md"]:::artifact
        DIAGRAMS["diagrams/*.md"]:::artifact
    end

    subgraph PlanifTickets
        C3["/plan-backend-ticket"]:::cmd
        C4["/plan-frontend-ticket"]:::cmd
        BD["backend-developer"]:::agent
        FD["frontend-developer"]:::agent
        BE_PLAN["Plan Backend"]:::artifact
        FE_PLAN["Plan Frontend"]:::artifact
    end

    subgraph Implementacion
        C5["/develop-backend"]:::cmd
        C6["/develop-frontend"]:::cmd
        CODE["Codigo App"]:::artifact
    end

    subgraph Versionado
        C7["/commit"]:::cmd
        GIT["Git Commit + PR"]:::external
    end

    subgraph Sincronizacion
        C8["/push-backlog-plan"]:::cmd
        JIRA["Jira / GitHub"]:::external
    end

    ProjectContext --> C1
    C1 -.-> PM_BA
    C1 --> PRD

    PRD --> C2
    PRD --> C2b
    C2 -.-> BP
    C2b -.-> DA
    C2b --> DIAGRAMS
    DIAGRAMS -.-> C2
    C2 --> BACKLOG
    C2 --> US

    US --> C3
    US --> C4
    C3 -.-> BD
    C4 -.-> FD
    C3 --> BE_PLAN
    C4 --> FE_PLAN

    BE_PLAN --> C5
    FE_PLAN --> C6
    C5 --> CODE
    C6 --> CODE

    CODE --> C7
    C7 --> GIT

    BACKLOG --> C8
    US --> C8
    DIAGRAMS -.-> C8
    C8 --> JIRA
```

---

## 4. Flujo Detallado por Fase

### Fase 1: Estrategia de Producto

**Comando**: `/create-prd`
**Agente**: `product-manager-ba.md`
**Entrada**: Archivo de contexto del proyecto
**Salida**: `PRD.md`

```mermaid
flowchart LR
    classDef input fill:#fff3cd,stroke:#ffc107,stroke-width:2px,color:#1e293b
    classDef step fill:#f8fafc,stroke:#94a3b8,stroke-width:2px,color:#1e293b
    classDef output fill:#f3e8ff,stroke:#9333ea,stroke-width:2px,color:#1e293b

    IN[("Contexto del proyecto")]:::input
    A[1. Analizar problema]:::step
    B[2. Definir vision]:::step
    C[3. Identificar usuarios]:::step
    D[4. Definir requisitos]:::step
    E[5. Disenar data model]:::step
    F[6. Definir arquitectura]:::step
    OUT[/PRD.md/]:::output

    IN --> A --> B --> C --> D --> E --> F --> OUT
```

---

### Fase 2: Backlog y Diagramas

**Comandos**: `/create-backlog-plan` + `/create-diagrams`
**Agentes**: `backlog-planner.md` + `diagram-architect.md`
**Entrada**: `PRD.md`
**Salida**: `backlog.md` + `us-XXX.md` + `diagrams/*.md`

```mermaid
flowchart LR
    classDef input fill:#fff3cd,stroke:#ffc107,stroke-width:2px,color:#1e293b
    classDef step fill:#f8fafc,stroke:#94a3b8,stroke-width:2px,color:#1e293b
    classDef output fill:#f3e8ff,stroke:#9333ea,stroke-width:2px,color:#1e293b

    PRD[("PRD.md")]:::input

    A[1. Descomponer en Epicas]:::step
    B[2. Generar Historias de Usuario]:::step
    C[3. Estimar Fibonacci]:::step
    D[4. Priorizar RICE]:::step
    E[5. Escribir BDD Gherkin]:::step
    F[6. Definir release plan]:::step

    OUT1[/backlog.md/]:::output
    OUT2[/us-XXX.md/]:::output
    OUT3[/diagrams/*.md/]:::output

    PRD --> A --> B --> C --> D --> E --> F
    F --> OUT1
    F --> OUT2
    PRD --> OUT3
```

**Contenido de cada Historia de Usuario:**

```mermaid
flowchart LR
    classDef req fill:#f8fafc,stroke:#94a3b8,stroke-width:2px,color:#1e293b
    classDef bdd fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#1e293b
    classDef tech fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e293b

    A["Formato: Como... quiero... para..."]:::req
    B["Estimacion: Fibonacci + T-Shirt"]:::req
    C["Prioridad: RICE + MoSCoW"]:::req
    D["Criterios BDD: 3+ escenarios Gherkin"]:::bdd
    E["Notas Tecnicas: archivos, endpoints"]:::tech
    F["Dependencias: bloqueado por / bloquea"]:::tech
    G["Definition of Done"]:::req

    A --> B --> C --> D --> E --> F --> G
```

---

### Fase 3: Mantenimiento del Backlog

**Comandos**: `/add-us`, `/update-us`, `/delete-us`, `/enrich-us`
**Regla**: Todos sincronizan tanto el archivo `us-XXX.md` como el `PRD.md`

```mermaid
flowchart LR
    classDef action fill:#fef3c7,stroke:#f59e0b,stroke-width:2px,color:#1e293b
    classDef artifact fill:#f3e8ff,stroke:#9333ea,stroke-width:2px,color:#1e293b
    classDef sync fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e293b

    ADD["/add-us crea nueva US"]:::action
    UPDATE["/update-us modifica US"]:::action
    DELETE["/delete-us elimina US"]:::action
    ENRICH["/enrich-us enriquece US"]:::action

    US["us-XXX.md"]:::artifact
    PRD["PRD.md"]:::artifact

    ADD --> US
    UPDATE --> US
    DELETE --> US
    ENRICH --> US
    ADD -.->|Sincroniza| PRD
    UPDATE -.->|Sincroniza| PRD
    DELETE -.->|Limpia referencias| PRD
```

---

### Fase 4: Planificacion de Tickets

**Comandos**: `/plan-backend-ticket` + `/plan-frontend-ticket`
**Agentes**: `backend-developer.md` + `frontend-developer.md`
**Entrada**: Ticket Jira o archivo US local
**Salida**: Plan de implementacion en `docs/plans/`

| Componente | Backend | Frontend |
|---|---|---|
| Stack | Node.js + Express + Prisma + PostgreSQL | React 18 + TypeScript + Bootstrap 5 |
| Arquitectura | DDD (Layered: Presentation - Application - Domain) | Componentes React + Service Layer |
| Salida | `docs/plans/[ID]_backend.md` | `docs/plans/[ID]_frontend.md` |
| Contenido | Branch name, file paths, function signatures, DDD steps, tests | Branch name, componentes, servicios, routing, tests Cypress |

```mermaid
flowchart TD
    classDef input fill:#fff3cd,stroke:#ffc107,stroke-width:2px,color:#1e293b
    classDef plan fill:#dbeafe,stroke:#2563eb,stroke-width:2px,color:#1e293b
    classDef step fill:#f8fafc,stroke:#94a3b8,stroke-width:2px,color:#1e293b

    TICKET[("Ticket Jira o US local")]:::input

    subgraph BackendPlan
        B1[Crear rama: feature/[ID]-backend]:::step
        B2[Implementar validation]:::step
        B3[Implementar service]:::step
        B4[Implementar controller]:::step
        B5[Agregar route]:::step
        B6[Escribir tests unitarios]:::step
        B7[Actualizar docs]:::step
    end

    subgraph FrontendPlan
        F1[Crear rama: feature/[ID]-frontend]:::step
        F2[Crear/actualizar service API]:::step
        F3[Crear/actualizar componentes]:::step
        F4[Actualizar routing]:::step
        F5[Escribir tests Cypress]:::step
        F6[Actualizar docs]:::step
    end

    TICKET -->|plan-backend-ticket| B1
    TICKET -->|plan-frontend-ticket| F1
    B1 --> B2 --> B3 --> B4 --> B5 --> B6 --> B7
    F1 --> F2 --> F3 --> F4 --> F5 --> F6
```

---

### Fase 5: Implementacion

**Comandos**: `/develop-backend` + `/develop-frontend`
**Salida**: Codigo funcionando con tests pasando

```mermaid
flowchart LR
    classDef step fill:#f8fafc,stroke:#94a3b8,stroke-width:2px,color:#1e293b
    classDef result fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#1e293b
    classDef check fill:#fef3c7,stroke:#f59e0b,stroke-width:2px,color:#1e293b

    PLAN[("Plan de implementacion")]

    A[1. Crear feature branch]:::step
    B[2. Implementar codigo]:::step
    C[3. Ejecutar linter + typecheck]:::check
    D[4. Escribir tests cobertura 90%]:::step
    E[5. Actualizar documentacion]:::step
    F[6. Stage solo archivos del ticket]:::step

    DONE["Codigo listo para commit"]:::result

    PLAN --> A --> B --> C
    C -.->|Falla| B
    C --> D --> E --> F --> DONE
```

---

### Fase 6: Commit y Pull Request

**Comando**: `/commit [Ticket-ID]`
**Salida**: Git commit + PR en GitHub

```mermaid
flowchart LR
    classDef step fill:#f8fafc,stroke:#94a3b8,stroke-width:2px,color:#1e293b
    classDef output fill:#2563eb,stroke:#1d4ed8,stroke-width:2px,color:#fff

    CODE[("Cambios stageados")]

    A[1. git commit con mensaje en ingles]:::step
    B[2. git push al remoto]:::step
    C[3. gh pr create]:::step

    PR["Pull Request en GitHub"]:::output

    CODE --> A --> B --> C --> PR
```

---

### Fase 7: Sincronizacion con Plataforma

**Comando**: `/push-backlog-plan jira|github`
**Salida**: Epicas y Stories en Jira o GitHub Issues

```mermaid
flowchart LR
    classDef input fill:#fff3cd,stroke:#ffc107,stroke-width:2px,color:#1e293b
    classDef step fill:#f8fafc,stroke:#94a3b8,stroke-width:2px,color:#1e293b
    classDef output fill:#dcfce7,stroke:#16a34a,stroke-width:2px,color:#1e293b

    BACKLOG[("backlog.md")]:::input
    US[("us-XXX.md")]:::input

    V[1. Validar que todas las US existen]:::step
    E[2. Crear Epicas en Jira]:::step
    S[3. Crear Stories con descripcion ADF]:::step
    D[4. Enlazar dependencias]:::step
    L[5. Agregar labels: mvp, v1.1]:::step
    U[6. Actualizar archivos locales con IDs]:::step

    EPICS["Epicas en Jira"]:::output
    STORIES["Stories en Jira"]:::output

    BACKLOG --> V
    US --> V
    V --> E --> S --> D --> L --> U
    E --> EPICS
    S --> STORIES
    U -.->|Guarda IDs| BACKLOG
    U -.->|Guarda IDs| US
```

---

## 5. Comandos Auxiliares

| Comando | Proposito |
|---|---|
| `/explain` | Ensenia el concepto detras de una pregunta usando modelos mentales y quiz |
| `/meta-prompt` | Refina y reestructura un prompt aplicando mejores practicas |
| `/update-docs` | Sincroniza documentacion segun documentation-standards.mdc |

---

## 6. Principios de Diseno Clave

1. **Fuente Unica de Verdad**: `PRD.md` es el artefacto raiz. Los comandos de mantenimiento siempre sincronizan de vuelta al PRD.

2. **Enriquecimiento Progresivo**: Cada comando agrega una capa de detalle sin reescribir capas anteriores:
   - `PRD.md` -> requisitos de negocio
   - `backlog/` -> historias con BDD
   - `diagrams/` -> documentacion visual
   - `docs/plans/` -> pasos de implementacion
   - Codigo -> software funcionando

3. **Consistencia Cruzada**: Comandos en fases posteriores leen artefactos de fases anteriores para asegurar alineacion.

4. **Separacion de Preocupaciones**: Backend y frontend tienen comandos independientes con sufijos `-backend` / `-frontend` para trabajo en paralelo.

5. **Documentacion como Entregable**: Cada implementacion debe actualizar la documentacion como paso final.

---

## 7. Resumen para la Demo

```mermaid
flowchart LR
    classDef f1 fill:#e1d5e7,stroke:#9673a6,stroke-width:3px,color:#1e293b
    classDef f2 fill:#dae8fc,stroke:#6c8ebf,stroke-width:3px,color:#1e293b
    classDef f3 fill:#d5e8d4,stroke:#82b366,stroke-width:3px,color:#1e293b
    classDef out fill:#ffe6cc,stroke:#d79b00,stroke-width:3px,color:#1e293b

    F1["F1: Estrategia"]:::f1
    F2["F2-3: Planeacion"]:::f2
    F3["F4-7: Ejecucion"]:::f3
    OUT["Codigo + Jira"]:::out

    F1 --> F2 --> F3 --> OUT
```

| Fase | Comandos | Entrada | Salida |
|---|---|---|---|
| **1. Estrategia** | `/create-prd` | Contexto del proyecto | `PRD.md` |
| **2. Backlog + Diagramas** | `/create-backlog-plan`, `/create-diagrams` | `PRD.md` | `backlog.md`, `us-XXX.md`, `diagrams/*.md` |
| **3. Mantenimiento** | `/add-us`, `/update-us`, `/delete-us`, `/enrich-us` | `us-XXX.md` + `PRD.md` | `us-XXX.md` actualizado |
| **4. Planificacion Tickets** | `/plan-backend-ticket`, `/plan-frontend-ticket` | Ticket Jira o US local | Plan en `docs/plans/` |
| **5. Implementacion** | `/develop-backend`, `/develop-frontend` | Plan en `docs/plans/` | Codigo + tests |
| **6. Commit + PR** | `/commit` | Cambios stageados | Git commit + PR |
| **7. Sincronizacion** | `/push-backlog-plan` | `backlog.md` + `us-XXX.md` | Epicas + Stories en Jira/GitHub |
