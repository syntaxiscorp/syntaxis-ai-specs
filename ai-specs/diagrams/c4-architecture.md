# C4 Architecture Diagrams: SMAT System

This file documents the system architecture at three zoom levels following the C4 model: System Context (who interacts with SMAT), Container (how the system is decomposed into deployable units), and Component (how the backend API is internally structured). These diagrams inform technology decisions and infrastructure planning.

---

## Level 1: System Context

Shows SMAT as a black box, its users (human actors), and the external systems it integrates with. This diagram is intended for non-technical stakeholders and senior management.

```mermaid
flowchart TB
    Admin(["👤 Administrador\n[Person]\nGestiona usuarios, obras y permisos"])
    Analyst(["👤 Analista\n[Person]\nCarga archivos XML semanales"])
    Validator(["👤 Validador\n[Person]\nValida avances por obra"])
    Controller(["👤 Controlador\n[Person]\nRegistra avances en terreno"])
    Manager(["👤 Gerente\n[Person]\nVisualiza el tablero ejecutivo"])

    subgraph SMAT_BOUNDARY ["🏗️ SMAT — Sistema de Control de Avance Físico de Obras"]
        SMAT["SMAT\n[Software System]\nCentraliza la gestión y control\nde avance físico de obras de construcción"]
    end

    subgraph EXTERNAL ["Sistemas Externos"]
        XMLSrc["📁 Archivos XML\n[External System]\nProgramas de obra exportados\ndesde MS Project u equivalente"]
        Email["📧 Servicio de Email\n[External System]\nEnvío de notificaciones\ny alertas a usuarios"]
        AuthProvider["🔐 Proveedor de Autenticación\n[External System — Futuro]\nSSO / LDAP corporativo"]
    end

    Admin -->|"Gestiona usuarios, obras\npermisos y perfiles\n[HTTPS]"| SMAT
    Analyst -->|"Carga archivos XML\ny habilita vistas\n[HTTPS]"| SMAT
    Validator -->|"Revisa y valida\navances semanales\n[HTTPS]"| SMAT
    Controller -->|"Registra avances\nen partidas\n[HTTPS]"| SMAT
    Manager -->|"Visualiza tablero\ny estado de obras\n[HTTPS]"| SMAT

    SMAT -->|"Recibe archivos\nde programa de obra\n[Upload]"| XMLSrc
    SMAT -->|"Envía alertas\ny notificaciones\n[SMTP / API]"| Email
    SMAT -.->|"Autenticación SSO\n[OAuth 2.0 — Futuro]"| AuthProvider

    style SMAT fill:#1168bd,color:#fff,stroke:#0b4884
    style XMLSrc fill:#999,color:#fff,stroke:#777
    style Email fill:#999,color:#fff,stroke:#777
    style AuthProvider fill:#ccc,color:#555,stroke:#aaa,stroke-dasharray:5 5
```

---

## Level 2: Container Diagram

Decomposes SMAT into its major deployable units and shows how they communicate. This diagram is intended for architects and senior developers.

```mermaid
flowchart TB
    Controller(["👤 Controlador / Validador\nAnalista / Admin / Gerente\n[Person]"])

    subgraph SMAT_SYS ["Sistema SMAT"]
        SPA["🖥️ Web Application\n[Container: SPA — React / Angular]\nInterfaz de usuario responsive.\nEjecutada en el navegador del usuario."]

        API["⚙️ Backend API\n[Container: REST API — Node.js / .NET]\nManeja toda la lógica de negocio,\nautenticación y autorización (RBAC).\nExpone endpoints REST/JSON."]

        XMLWorker["🔄 XML Processor\n[Container: Background Worker]\nProcesa archivos XML de forma asíncrona.\nActualiza ESTRUCTURA_OBRA y\nNODO_ESTADO_SEMANAL."]

        Queue["📬 Job Queue\n[Container: Redis / RabbitMQ]\nCola de trabajos para procesamiento\nasíncrono de archivos XML."]

        DB[("🗄️ Database\n[Container: PostgreSQL]\nAlmacena todos los datos\ndel sistema: obras, usuarios,\ncargas, avances, validaciones.")]

        Cache["⚡ Cache\n[Container: Redis]\nAlmacena resultados de\nconsultas frecuentes\n(tablero, vistas de control)."]

        Storage["📦 File Storage\n[Container: S3 / Azure Blob]\nAlmacena los archivos XML\ncargados y los archivos\ndisponibles para descarga."]
    end

    Email["📧 Email Service\n[External System]"]

    Controller -->|"Usa la aplicación\n[HTTPS]"| SPA
    SPA -->|"Llamadas REST\n[JSON / HTTPS]"| API
    API -->|"Lee y escribe\n[SQL / ORM]"| DB
    API -->|"Lee y escribe\ncaché\n[Redis Protocol]"| Cache
    API -->|"Encola trabajos\nXML\n[AMQP / Redis]"| Queue
    API -->|"Sube y descarga\narchivos\n[S3 API / HTTPS]"| Storage
    Queue -->|"Consume trabajos"| XMLWorker
    XMLWorker -->|"Actualiza estructura\nde obra\n[SQL / ORM]"| DB
    XMLWorker -->|"Lee archivos\na procesar\n[S3 API]"| Storage
    API -->|"Envía notificaciones\n[SMTP / API]"| Email

    style SPA fill:#23a9f2,color:#fff,stroke:#1a7bbf
    style API fill:#1168bd,color:#fff,stroke:#0b4884
    style XMLWorker fill:#1168bd,color:#fff,stroke:#0b4884
    style DB fill:#e8a838,color:#fff,stroke:#c4861c
    style Queue fill:#e8a838,color:#fff,stroke:#c4861c
    style Cache fill:#e8a838,color:#fff,stroke:#c4861c
    style Storage fill:#e8a838,color:#fff,stroke:#c4861c
    style Email fill:#999,color:#fff,stroke:#777
```

---

## Level 3: Component Diagram — Backend API

Shows the internal components of the Backend API container, illustrating the layered architecture and how each component fulfills a specific responsibility.

```mermaid
flowchart TB
    SPA["🖥️ Web Application\n[Container]"]
    DB[("🗄️ PostgreSQL\n[Container]")]
    Cache["⚡ Redis Cache\n[Container]"]
    Storage["📦 File Storage\n[Container]"]
    Queue["📬 Job Queue\n[Container]"]

    subgraph API_BOUNDARY ["⚙️ Backend API [Container]"]
        subgraph HTTP ["HTTP Layer"]
            Router["🔀 Router\n[Component: Express / .NET]\nEnruta peticiones HTTP\na los controladores correctos"]
            AuthMW["🔐 Auth Middleware\n[Component]\nVerifica JWT y aplica\nRBAC por rol y obra"]
        end

        subgraph Controllers ["Controllers Layer"]
            UserCtrl["👤 Users Controller\n[Component]\nGestión de usuarios\ny asignaciones"]
            ObraCtrl["🏗️ Obras Controller\n[Component]\nGestión de obras\ny sub-etapas"]
            CargaCtrl["📁 Cargas Controller\n[Component]\nCarga de archivos\nXML semanales"]
            ControlCtrl["📊 Control Controller\n[Component]\nVistas Gantt/PTS/Física\ny registro de avances"]
            ValCtrl["✅ Validación Controller\n[Component]\nCierres de control\ny validación"]
            DashCtrl["📈 Dashboard Controller\n[Component]\nTablero gerencial\ny alertas"]
        end

        subgraph Services ["Application Services Layer"]
            UserSvc["UserService\n[Component]\nLógica de negocio\nde usuarios"]
            ObraSvc["ObraService\n[Component]\nLógica de negocio\nde obras"]
            CargaSvc["CargaService\n[Component]\nValidación de ventana\nhoraria y encolado XML"]
            ProgressSvc["ProgressService\n[Component]\nRegistro y consulta\nde avances"]
            ClosureSvc["ClosureService\n[Component]\nCierre de control,\ncierre de validación\ny cierre automático"]
            DashSvc["DashboardService\n[Component]\nAgregación de\nmétricas del portafolio"]
        end

        subgraph Repos ["Repository Layer"]
            UserRepo["UserRepository\n[Component: ORM]\nAcceso a datos\nde usuarios y perfiles"]
            ObraRepo["ObraRepository\n[Component: ORM]\nAcceso a datos\nde obras"]
            CargaRepo["CargaRepository\n[Component: ORM]\nAcceso a datos\nde cargas"]
            AvanceRepo["AvanceRepository\n[Component: ORM]\nAcceso a datos\nde avances y lotes"]
            ValRepo["ValidacionRepository\n[Component: ORM]\nAcceso a datos\nde validaciones"]
        end
    end

    SPA -->|"HTTP Request"| Router
    Router --> AuthMW
    AuthMW --> UserCtrl
    AuthMW --> ObraCtrl
    AuthMW --> CargaCtrl
    AuthMW --> ControlCtrl
    AuthMW --> ValCtrl
    AuthMW --> DashCtrl

    UserCtrl --> UserSvc
    ObraCtrl --> ObraSvc
    CargaCtrl --> CargaSvc
    ControlCtrl --> ProgressSvc
    ValCtrl --> ClosureSvc
    DashCtrl --> DashSvc

    UserSvc --> UserRepo
    ObraSvc --> ObraRepo
    CargaSvc --> CargaRepo
    CargaSvc -->|"Encola trabajo"| Queue
    CargaSvc -->|"Sube archivo"| Storage
    ProgressSvc --> AvanceRepo
    ProgressSvc --> ObraRepo
    ClosureSvc --> AvanceRepo
    ClosureSvc --> ValRepo
    DashSvc -->|"Lee caché"| Cache
    DashSvc --> ObraRepo

    UserRepo --> DB
    ObraRepo --> DB
    CargaRepo --> DB
    AvanceRepo --> DB
    ValRepo --> DB

    style Router fill:#23a9f2,color:#fff,stroke:#1a7bbf
    style AuthMW fill:#e05d0a,color:#fff,stroke:#b84a07
    style UserCtrl fill:#1168bd,color:#fff,stroke:#0b4884
    style ObraCtrl fill:#1168bd,color:#fff,stroke:#0b4884
    style CargaCtrl fill:#1168bd,color:#fff,stroke:#0b4884
    style ControlCtrl fill:#1168bd,color:#fff,stroke:#0b4884
    style ValCtrl fill:#1168bd,color:#fff,stroke:#0b4884
    style DashCtrl fill:#1168bd,color:#fff,stroke:#0b4884
    style DB fill:#e8a838,color:#fff,stroke:#c4861c
    style Cache fill:#e8a838,color:#fff,stroke:#c4861c
    style Storage fill:#e8a838,color:#fff,stroke:#c4861c
    style Queue fill:#e8a838,color:#fff,stroke:#c4861c
```

---

## Notes & Assumptions

- **Technology stack**: The PRD recommends but does not lock in a specific stack. These diagrams use Node.js / .NET for the API and React/Angular for the SPA as representative choices aligned with the PRD's architecture overview.
- **XML Processing**: The async Worker + Queue pattern is a design recommendation to handle large XML files without blocking the API; if file sizes are consistently small (< 1MB), a synchronous approach within the API may suffice.
- **Cache layer**: Used primarily for the Dashboard and Control views, which aggregate data across many entities. Cache invalidation should occur on each new validation closure.
- **SSO / Auth Provider**: Shown as a future integration (dashed line in Context diagram); current implementation assumes local JWT-based authentication.
- **Level 4 (Code)** diagrams are not included in this version; they should be generated once the backend codebase is established, particularly for `ClosureService` (complex business logic) and `XMLParser` (critical processing component).
