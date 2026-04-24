# Sequence Diagrams: SMAT System

This file contains one sequence diagram per major system flow, covering authentication, weekly file upload, progress registration, control closure, and validation closure. These diagrams map directly to the API endpoints defined in the PRD and the use cases in `use-cases.md`.

---

## 1. Authentication Flow

A user enters credentials and receives a JWT token used for all subsequent API calls. Invalid credentials return a 401 without a token.

```mermaid
sequenceDiagram
    actor User
    participant Frontend
    participant API as API REST
    participant AuthSvc as Auth Service
    participant DB as Database

    User->>Frontend: Ingresa email + contraseña
    Frontend->>API: POST /api/auth/login { email, password }
    API->>AuthSvc: validateCredentials(email, password)
    AuthSvc->>DB: SELECT usuario WHERE email = ?
    DB-->>AuthSvc: Registro de usuario (hash contraseña)

    alt Credenciales válidas
        AuthSvc->>DB: SELECT perfiles y permisos del usuario
        DB-->>AuthSvc: Perfiles activos
        AuthSvc-->>API: { userId, perfiles, permisos }
        API-->>Frontend: 200 OK { accessToken, refreshToken }
        Frontend-->>User: Redirige al tablero (/)
    else Credenciales inválidas
        AuthSvc-->>API: AuthenticationError
        API-->>Frontend: 401 Unauthorized
        Frontend-->>User: Muestra mensaje de error
    end
```

---

## 2. Weekly File Upload (Analyst)

The Analyst uploads an XML file for a project within the Thursday-to-Thursday 08:00 window. The system validates the time window, validates the file format, and processes the project structure asynchronously.

```mermaid
sequenceDiagram
    actor Analyst as Analista
    participant Frontend
    participant API as API REST
    participant TimeSvc as Validación\nVentana Horaria
    participant XMLParser as Procesador XML
    participant DB as Database
    participant Storage as Almacenamiento\nArchivos

    Analyst->>Frontend: Selecciona obra y adjunta archivo XML
    Frontend->>API: POST /api/obras/{obraId}/cargas (multipart/form-data)
    API->>TimeSvc: isWithinUploadWindow(now)

    alt Fuera de la ventana horaria (no entre jueves-jueves 08:00)
        TimeSvc-->>API: WindowClosedError
        API-->>Frontend: 422 Unprocessable Entity { error: "Fuera de ventana de carga" }
        Frontend-->>Analyst: Muestra error con próxima ventana disponible
    else Dentro de la ventana
        TimeSvc-->>API: OK
        API->>XMLParser: validateXMLSchema(file)

        alt XML inválido
            XMLParser-->>API: ValidationError { details }
            API-->>Frontend: 400 Bad Request { errors }
            Frontend-->>Analyst: Muestra errores de formato
        else XML válido
            XMLParser-->>API: SchemaValid
            API->>Storage: upload(file) → fileUrl
            Storage-->>API: fileUrl
            API->>DB: INSERT CARGAS { obraId, estado: PROCESANDO, nombreArchivo, fechaInicio }
            DB-->>API: cargaId
            API-->>Frontend: 202 Accepted { cargaId, estado: "procesando" }
            Frontend-->>Analyst: Muestra estado "Procesando..."

            Note over API, DB: Procesamiento asíncrono
            API->>XMLParser: parseAndBuildStructure(fileUrl, obraId, cargaId)
            XMLParser->>DB: UPSERT ESTRUCTURA_OBRA (nodos del programa)
            XMLParser->>DB: INSERT NODO_ESTADO_SEMANAL (estado semanal por nodo)
            XMLParser->>DB: UPDATE CARGAS SET estado = PROCESADO
            DB-->>XMLParser: OK

            Frontend->>API: GET /api/cargas/{cargaId}/estado (polling)
            API-->>Frontend: 200 OK { estado: "procesado" }
            Frontend-->>Analyst: "Carga completada exitosamente"
        end
    end
```

---

## 3. Progress Registration (Controller)

A Controller selects a view (Gantt / PTS / Física), navigates to a work item node, and registers a progress percentage with an optional observation. The system saves the record to the controller's open batch.

```mermaid
sequenceDiagram
    actor Controller as Controlador
    participant Frontend
    participant API as API REST
    participant AuthMW as Auth Middleware\n(RBAC)
    participant ProgSvc as Progress Service
    participant DB as Database

    Controller->>Frontend: Accede a Control de Avances (elige vista: Gantt/PTS/Física)
    Frontend->>API: GET /api/obras/{obraId}/control/{vista}
    API->>AuthMW: checkAssignment(userId, obraId, rol: CONTROLADOR)

    alt Sin asignación activa en la obra
        AuthMW-->>API: ForbiddenError
        API-->>Frontend: 403 Forbidden
        Frontend-->>Controller: "No tienes acceso a esta obra"
    else Asignación válida
        AuthMW-->>API: OK { usuarioObraPerfilId }
        API->>DB: SELECT estructura + nodo_estado_semanal WHERE obraId + semana actual
        DB-->>API: Nodos del programa con estado semanal
        API-->>Frontend: 200 OK { nodos, semana, loteId }
        Frontend-->>Controller: Muestra tabla de partidas con % avance actual

        Controller->>Frontend: Ingresa % avance + observación en partida (nodoId)
        Frontend->>API: POST /api/lotes/{loteId}/avances { nodoEstadoSemanalId, porcentaje, observacion }
        API->>AuthMW: checkLoteOwnership(userId, loteId)
        AuthMW-->>API: OK

        API->>ProgSvc: registerProgress(loteId, nodoEstadoSemanalId, porcentaje, observacion, usuarioObraPerfilId)
        ProgSvc->>DB: SELECT nodo — check if already 100% by another controller
        
        alt Partida ya completada por otro controlador
            DB-->>ProgSvc: avance existente 100%
            ProgSvc-->>API: WorkItemAlreadyCompleteError
            API-->>Frontend: 409 Conflict { message: "Partida ya terminada" }
            Frontend-->>Controller: Muestra partida como terminada (solo lectura)
        else Partida disponible
            DB-->>ProgSvc: nodo disponible
            ProgSvc->>DB: INSERT AVANCES { loteAvanceId, nodoEstadoSemanalId, observacion, usuarioObraPerfilId, fechaRegistro }
            DB-->>ProgSvc: avanceId
            ProgSvc-->>API: { avanceId }
            API-->>Frontend: 201 Created { avanceId }
            Frontend-->>Controller: Actualiza % en la tabla en tiempo real
        end
    end
```

---

## 4. Control Closure (Controller)

A Controller confirms they have finished entering progress for the week. The system closes their batch permanently. This action is irreversible.

```mermaid
sequenceDiagram
    actor Controller as Controlador
    participant Frontend
    participant API as API REST
    participant ClosureSvc as Closure Service
    participant AutoValSvc as Auto-Validation\nService
    participant DB as Database

    Controller->>Frontend: Hace clic en "Cerrar Control"
    Frontend->>Controller: Solicita confirmación ("¿Estás seguro?")
    Controller->>Frontend: Confirma

    Frontend->>API: POST /api/lotes/{loteId}/cierre
    API->>DB: SELECT lote WHERE loteId — check estado
    
    alt Lote ya cerrado
        DB-->>API: lote.fechaCierre != null
        API-->>Frontend: 409 Conflict { message: "Lote ya cerrado" }
        Frontend-->>Controller: "El control ya fue cerrado"
    else Lote abierto
        DB-->>API: lote abierto
        API->>ClosureSvc: closeControlBatch(loteId)
        ClosureSvc->>DB: UPDATE LOTE_AVANCES SET fechaCierre = NOW() WHERE loteId
        DB-->>ClosureSvc: OK
        ClosureSvc-->>API: { fechaCierre }
        API-->>Frontend: 200 OK { estado: "cerrado", fechaCierre }
        Frontend-->>Controller: "Control cerrado exitosamente"

        Note over API, AutoValSvc: Verificación de cierre automático
        API->>AutoValSvc: checkAllControllersClosedForWeek(obraId, semana)
        AutoValSvc->>DB: SELECT lotes WHERE obraId + semana — count abiertos
        
        alt Todos los controladores cerraron
            DB-->>AutoValSvc: 0 lotes abiertos
            AutoValSvc->>DB: INSERT VALIDACION_SEMANA { estado: APROBADO_AUTOMATICO, fechaValidacion: NOW() }
            DB-->>AutoValSvc: validacionId
            AutoValSvc->>DB: INSERT VALIDACION_LOTE para cada lote
            AutoValSvc-->>API: { validacionId, tipo: "automatica" }
            Note over API: Notifica al Validador (alerta futura)
        else Aún hay controladores abiertos
            DB-->>AutoValSvc: N lotes abiertos
            AutoValSvc-->>API: { pendientes: N }
            Note over API: No se dispara cierre automático
        end
    end
```

---

## 5. Manual Validation Closure (Validator)

The Validator reviews the progress submitted by all controllers for the week, checks that all have closed their control, and manually executes the validation closure. After this, files become available for download.

```mermaid
sequenceDiagram
    actor Validator as Validador
    participant Frontend
    participant API as API REST
    participant AuthMW as Auth Middleware
    participant ValSvc as Validation Service
    participant DB as Database
    participant Storage as Almacenamiento\nArchivos

    Validator->>Frontend: Accede al Panel de Validación de la obra
    Frontend->>API: GET /api/obras/{obraId}/validaciones?semana=actual
    API->>AuthMW: checkRole(userId, obraId, rol: VALIDADOR)
    AuthMW-->>API: OK

    API->>DB: SELECT lotes + controladores + estado cierre WHERE obraId + semana
    DB-->>API: Lista de lotes con fechaCierre por controlador
    API-->>Frontend: 200 OK { controladores: [{nombre, estado: abierto|cerrado}] }
    Frontend-->>Validator: Muestra tabla de controladores con estado

    alt Hay controladores sin cerrar
        Frontend-->>Validator: Botón "Cerrar Validación" deshabilitado\n+ indicador de controladores pendientes
        Note over Validator, Frontend: Validador debe esperar o coordinar con controladores
    else Todos cerraron control
        Frontend-->>Validator: Botón "Cerrar Validación" habilitado

        Validator->>Frontend: Hace clic en "Cerrar Validación" + ingresa observación opcional
        Frontend->>API: POST /api/validaciones/{validacionId}/cierre { observacion }
        API->>AuthMW: checkRole(userId, obraId, rol: VALIDADOR)
        API->>ValSvc: performValidationClosure(obraId, semana, observacion, validadorId)

        ValSvc->>DB: INSERT VALIDACION_SEMANA { loteAvanceId, usuarioValidadorId, estadoValidacionId: APROBADO, fechaValidacion, observacion }
        DB-->>ValSvc: validacionId

        ValSvc->>DB: INSERT VALIDACION_LOTE para cada lote de la semana
        DB-->>ValSvc: OK

        ValSvc->>Storage: markFilesAsDownloadable(cargaId)
        Storage-->>ValSvc: OK

        ValSvc-->>API: { validacionId, fechaValidacion, archivoDisponible: true }
        API-->>Frontend: 200 OK { validacionId, fechaValidacion }
        Frontend-->>Validator: "Validación cerrada. Archivos disponibles para descarga."
    end
```

---

## 6. File Download (Post-Validation)

A user with the appropriate role attempts to download the week's file. The system verifies that the validation closure exists before serving the download URL.

```mermaid
sequenceDiagram
    actor User as Usuario (Validador/Analista/Admin)
    participant Frontend
    participant API as API REST
    participant DB as Database
    participant Storage as Almacenamiento\nArchivos

    User->>Frontend: Hace clic en "Descargar" para una carga
    Frontend->>API: GET /api/cargas/{cargaId}/descarga
    API->>DB: SELECT validacion WHERE cargaId — check cierre validación

    alt Sin cierre de validación
        DB-->>API: validacion = null
        API-->>Frontend: 403 Forbidden { message: "Descarga disponible solo post-validación" }
        Frontend-->>User: Muestra estado "Pendiente de validación"
    else Validación cerrada
        DB-->>API: validacion.estadoValidacionId = APROBADO
        API->>Storage: generatePresignedUrl(fileKey, ttl: 15min)
        Storage-->>API: presignedUrl
        API-->>Frontend: 200 OK { downloadUrl }
        Frontend-->>User: Descarga inicia automáticamente
    end
```

---

## Notes & Assumptions

- All API calls use `Authorization: Bearer {jwt}` headers — this is omitted from diagrams for readability.
- Asynchronous XML processing (Diagram 2) assumes a job queue mechanism; the polling interval from frontend is assumed to be every 3–5 seconds.
- Auto-validation check (Diagram 4) is triggered synchronously after each control closure; in high-load scenarios this could be moved to an event-driven approach.
- `loteId` is created automatically when a Controller first accesses the control module for a given week; this initial creation call is omitted from Diagram 3 for brevity.
- The Validator role restriction (cannot close their own control as a Controller) is enforced at the `Auth Middleware` layer via the `USUARIO_OBRA_PERFIL` record.
- Presigned URLs for file download have a 15-minute TTL — this value should be confirmed with security requirements.
