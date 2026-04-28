# Product Backlog: Sistema SMAT

## Sync Status
- **Platform**: jira
- **Last Synced**: 2026-04-28
- **Epics Created**: 7 — SMAT-1, SMAT-6, SMAT-7, SMAT-8, SMAT-9, SMAT-66, SMAT-75
- **Stories Created**: 21 — SMAT-2, SMAT-10, SMAT-11, SMAT-12, SMAT-13, SMAT-14, SMAT-15, SMAT-16, SMAT-17, SMAT-18, SMAT-19, SMAT-20, SMAT-21, SMAT-22, SMAT-62, SMAT-67, SMAT-68, SMAT-76, SMAT-77, SMAT-78, SMAT-79
- **Stories Pending**: 0

## Metadata
- **PRD Fuente**: `ai-specs/specs/PRD.md`
- **Generado**: 2026-04-23
- **Método de Priorización**: RICE Scoring + MoSCoW
- **Método de Estimación**: Fibonacci + Planning Poker + T-Shirt
- **Total Historias**: 21
- **Total Story Points**: 123

---

## Alcance MVP

El MVP incluye todas las funcionalidades **Must Have** necesarias para operar el ciclo completo de control de avance físico semanal: autenticación, **integración de catálogos geográficos** (regiones y comunas desde BD comunes SALFA), gestión de acceso (usuarios, perfiles, permisos, asignaciones), gestión de obras, carga semanal de archivos XML, **administración centralizada de cargas por obra**, registro de avances (vistas Gantt y PTS), cierre de control por controlador, cierre de validación (manual y automático), y tablero ejecutivo. Incluye además la **aplicación móvil** (IONIC + Angular 20) para que los usuarios de obra (Controlador/Validador) puedan registrar avances y cerrar control/validación desde iOS y Android con soporte offline.

Las funcionalidades **Should Have** (vista Física, tipos/estados de obra, cierre automático de validación) se incluyen en Fase 2 por ser mejoras incrementales sobre una base funcional ya entregada.

---

## Resumen de Épicas

### Épica 1: Autenticación
- **Descripción**: Gestión del ciclo de autenticación de usuarios (login, logout, renovación de token JWT).
- **Objetivo de Negocio**: OB-5 — Control de acceso al sistema.
- **Historias**: US-001
- **Total Puntos**: 3
- **Prioridad**: Must Have

### Épica 2: Acceso y Seguridad
- **Descripción**: Gestión completa de usuarios, perfiles, permisos y asignación de usuarios a obras con roles específicos. Incluye tablero enriquecido con filtros avanzados para el Administrador.
- **Objetivo de Negocio**: OB-5 — Control de acceso basado en roles por obra.
- **Historias**: US-002, US-003, US-004, US-014
- **Total Puntos**: 16
- **Prioridad**: Must Have

### Épica 3: Obras
- **Descripción**: Creación y administración de obras, sub-etapas, tipos y estados de obra.
- **Objetivo de Negocio**: OB-1 — Centralizar gestión del portafolio de obras.
- **Historias**: US-005, US-006, US-007
- **Total Puntos**: 13
- **Prioridad**: Must / Should Have

### Épica 4: Cargas
- **Descripción**: Carga semanal de archivos XML con validación de ventana horaria, formato y procesamiento asíncrono. Incluye el módulo centralizado de administración de cargas por obra con acciones contextuales (cargar, desactivar vista física, descargar, bloquear tareas).
- **Objetivo de Negocio**: OB-2 — Reducir tiempo de cierre y validación semanal.
- **Historias**: US-008, US-009, US-015
- **Total Puntos**: 23
- **Prioridad**: Must / Should Have

### Épica 5: Control de Avances
- **Descripción**: Vistas de control (Gantt, PTS, Física), registro de avances por partida, cierre de control, cierre de validación y descarga post-validación.
- **Objetivo de Negocio**: OB-2, OB-3 — Cierre semanal en tiempo y trazabilidad completa.
- **Historias**: US-010, US-011, US-012, US-013
- **Total Puntos**: 31
- **Prioridad**: Must / Should Have

### Épica 6: Integración Comunas y Regiones
- **Descripción**: Integración con la base de datos de comunes de SALFA para exponer catálogos de regiones y comunas como servicios reutilizables. Datos de solo lectura desde tablas existentes en la BD corporativa. Incluye selectores en cascada Región → Comuna para formularios de obras y usuarios.
- **Objetivo de Negocio**: OB-1 — Estandarizar datos geográficos en el portafolio de obras.
- **Historias**: US-016, US-017
- **Total Puntos**: 6
- **Prioridad**: Must Have

### Épica 7: Aplicación Móvil
- **Descripción**: App móvil en IONIC con Angular 20 para usuarios de obra (Controlador y Validador). Permite registrar avances semanales y realizar el cierre de control y validación desde iOS y Android, con soporte offline y sincronización automática. Autenticación vía SSO Salfa.
- **Objetivo de Negocio**: OB-2, OB-3 — Habilitar el registro de avances y cierre en campo, sin dependencia del sistema web.
- **Historias**: US-018, US-019, US-020, US-021
- **Total Puntos**: 23
- **Prioridad**: Must Have

---

## Backlog Priorizado

| Rank | ID     | Título                                        | Épica               | Puntos | T-Shirt | MoSCoW  | Dependencias            |
|------|--------|-----------------------------------------------|---------------------|--------|---------|---------|-------------------------|
| 1    | US-001 | Inicio de Sesión                              | Autenticación                    | 3      | S       | Must    | —                       |
| 2    | US-016 | Listado de Regiones                           | Integración Comunas y Regiones   | 3      | S       | Must    | US-001                  |
| 3    | US-017 | Listado de Comunas                            | Integración Comunas y Regiones   | 3      | S       | Must    | US-016                  |
| 4    | US-002 | Gestión de Perfiles y Permisos                | Acceso y Seguridad               | 5      | M       | Must    | US-001                  |
| 5    | US-003 | Gestión de Usuarios                           | Acceso y Seguridad               | 5      | M       | Must    | US-001, US-002          |
| 6    | US-004 | Asignación de Usuario a Obra                  | Acceso y Seguridad               | 3      | S       | Must    | US-003, US-005          |
| 7    | US-014 | Tablero de Usuarios con Filtros Avanzados     | Acceso y Seguridad               | 3      | S       | Must    | US-003, US-004          |
| 8    | US-005 | Gestión de Obras                              | Obras                            | 8      | L       | Must    | US-001                  |
| 9    | US-006 | Gestión de Sub-Etapas                         | Obras                            | 3      | S       | Must    | US-005                  |
| 10   | US-008 | Carga Semanal de Archivo XML                  | Cargas                           | 13     | XL      | Must    | US-004, US-005          |
| 11   | US-015 | Administración de Cargas Semanales            | Cargas                           | 8      | L       | Must    | US-008                  |
| 12   | US-010 | Control de Avances — Vistas Gantt y PTS       | Control de Avances               | 13     | XL      | Must    | US-008                  |
| 13   | US-012 | Cierre de Control por Controlador             | Control de Avances               | 5      | M       | Must    | US-010                  |
| 14   | US-013 | Cierre de Validación (Manual y Automático)    | Control de Avances               | 8      | L       | Must    | US-012                  |
| 15   | US-007 | Gestión de Tipos y Estados de Obra            | Obras                            | 2      | XS      | Should  | US-001                  |
| 16   | US-009 | Habilitar Vista Física                        | Cargas                           | 2      | XS      | Should  | US-008                  |
| 17   | US-011 | Control de Avances — Vista Física             | Control de Avances               | 5      | M       | Should  | US-009, US-010          |
| 18   | US-018 | Autenticación Mobile vía SSO Salfa            | Aplicación Móvil                 | 5      | M       | Must    | US-001                  |
| 19   | US-019 | Mobile — Registro de Avances (Controlador)    | Aplicación Móvil                 | 8      | L       | Must    | US-018, US-010          |
| 20   | US-020 | Mobile — Cierre de Control (Controlador)      | Aplicación Móvil                 | 5      | M       | Must    | US-019, US-012          |
| 21   | US-021 | Mobile — Cierre de Validación (Validador)     | Aplicación Móvil                 | 5      | M       | Must    | US-020, US-013          |

---

## Plan de Lanzamiento

### Fase 1 — MVP (Sprints 1–10)
**Historias**: US-001, US-016, US-017, US-002, US-003, US-004, US-014, US-005, US-006, US-008, US-015, US-010, US-012, US-013, US-018, US-019, US-020, US-021  
**Puntos totales**: 103  
**Sprints estimados**: 10 sprints de ~10 puntos promedio  
**Resultado**: Sistema web operativo para el ciclo semanal de control de avance físico + app móvil (iOS/Android) con registro de avances, cierre de control y validación en campo.

### Fase 2 — v1.1 (Sprints 11–12)
**Historias**: US-007, US-009, US-011  
**Puntos totales**: 9  
**Resultado**: Vista Física habilitada, gestión de catálogos de tipos y estados de obra.

---

## Mapa de Dependencias

```
US-001 (Login)
  └── US-016 (Regiones) ← BD comunes SALFA
        └── US-017 (Comunas) ← BD comunes SALFA, filtro por región
  └── US-002 (Perfiles/Permisos)
        └── US-003 (Usuarios)
              └── US-004 (Asignación Usuario-Obra)
                    ├── US-014 (Tablero de Usuarios)
                    └── US-008 (Carga XML)
                          ├── US-015 (Administración de Cargas) ← también depende de US-009, US-013
                          └── US-010 (Vistas Gantt/PTS)
                                └── US-012 (Cierre Control)
                                      └── US-013 (Cierre Validación)
                                            └── US-015 (canDownloadActivo)
  └── US-005 (Obras) ← usa selectores US-016 y US-017
        ├── US-006 (Sub-Etapas)
        └── US-007 (Tipos/Estados)
  └── US-009 (Habilitar Vista Física) ← depende de US-008
        └── US-011 (Vista Física) ← también depende de US-010

[App Móvil]
US-018 (Auth Mobile) ← depende de SSO Salfa (US-001)
  └── US-019 (Registro Avances Mobile) ← también depende de US-010 (endpoints)
        └── US-020 (Cierre Control Mobile) ← también depende de US-012 (endpoints)
              └── US-021 (Cierre Validación Mobile) ← también depende de US-013 (endpoints)
```

---

## Riesgos y Supuestos

| Riesgo | Impacto | Mitigación |
|--------|---------|------------|
| US-008 y US-010 son las historias más complejas (13 pts c/u). Si el formato XML varía entre proyectos, el parser puede requerir más esfuerzo. | Alto | Definir y documentar el esquema XSD antes de iniciar US-008. |
| La regla de ventana horaria (jueves-jueves 08:00) requiere manejo de zona horaria (America/Santiago). | Medio | Implementar validación de timezone desde el inicio en US-008. |
| El cierre automático de validación (US-013) agrega complejidad al flujo de cierre de control (US-012). | Medio | Implementar US-013 como extensión de US-012, con cobertura de tests end-to-end. |
| La restricción de un solo Validador activo por obra requiere lógica de unicidad en US-004. | Medio | Agregar constraint de base de datos y validación en capa de servicio. |
| La BD de comunes de SALFA (US-016, US-017) es un sistema externo de solo lectura. Si cambia su esquema o está no disponible, los selectores de Región/Comuna fallarán. | Medio | Configurar caché IMemoryCache TTL 60 min + manejo de 503 explícito. Documentar contrato del esquema con equipo SALFA. |
| La app móvil (US-018–US-021) depende del comportamiento de deep links OAuth en iOS y Android, que puede variar entre versiones de OS. | Medio | Testear en dispositivos físicos iOS 16+ y Android 10+ desde el inicio. Usar @capacitor/browser (no cordova). |
| El modo offline de US-019 puede generar conflictos si el mismo nodo es avanzado desde la web y la app simultáneamente. | Medio | Implementar política last-write-wins con timestamp en el servidor. Documentar la restricción de uso concurrente. |

