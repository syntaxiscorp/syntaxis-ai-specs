# Product Backlog: Sistema SMAT

## Sync Status
- **Platform**: jira
- **Last Synced**: 2026-04-28
- **Epics Created**: 5 — SMAT-1, SMAT-6, SMAT-7, SMAT-8, SMAT-9
- **Stories Created**: 14 — SMAT-2, SMAT-10, SMAT-11, SMAT-12, SMAT-13, SMAT-14, SMAT-15, SMAT-16, SMAT-17, SMAT-18, SMAT-19, SMAT-20, SMAT-21, SMAT-22
- **Stories Pending**: 0

## Metadata
- **PRD Fuente**: `ai-specs/specs/PRD.md`
- **Generado**: 2026-04-23
- **Método de Priorización**: RICE Scoring + MoSCoW
- **Método de Estimación**: Fibonacci + Planning Poker + T-Shirt
- **Total Historias**: 15
- **Total Story Points**: 86

---

## Alcance MVP

El MVP incluye todas las funcionalidades **Must Have** necesarias para operar el ciclo completo de control de avance físico semanal: autenticación, gestión de acceso (usuarios, perfiles, permisos, asignaciones), gestión de obras, carga semanal de archivos XML, registro de avances (vistas Gantt y PTS), cierre de control por controlador, cierre de validación (manual y automático), y tablero ejecutivo. Estas funcionalidades cubren el flujo completo de negocio sin el cual el sistema no tiene valor operacional.

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
- **Descripción**: Carga semanal de archivos XML con validación de ventana horaria, formato y procesamiento asíncrono.
- **Objetivo de Negocio**: OB-2 — Reducir tiempo de cierre y validación semanal.
- **Historias**: US-008, US-009
- **Total Puntos**: 15
- **Prioridad**: Must / Should Have

### Épica 5: Control de Avances
- **Descripción**: Vistas de control (Gantt, PTS, Física), registro de avances por partida, cierre de control, cierre de validación y descarga post-validación.
- **Objetivo de Negocio**: OB-2, OB-3 — Cierre semanal en tiempo y trazabilidad completa.
- **Historias**: US-010, US-011, US-012, US-013
- **Total Puntos**: 31
- **Prioridad**: Must / Should Have

---

## Backlog Priorizado

| Rank | ID     | Título                                        | Épica               | Puntos | T-Shirt | MoSCoW  | Dependencias            |
|------|--------|-----------------------------------------------|---------------------|--------|---------|---------|-------------------------|
| 1    | US-001 | Inicio de Sesión                              | Autenticación       | 3      | S       | Must    | —                       |
| 2    | US-002 | Gestión de Perfiles y Permisos                | Acceso y Seguridad  | 5      | M       | Must    | US-001                  |
| 3    | US-003 | Gestión de Usuarios                           | Acceso y Seguridad  | 5      | M       | Must    | US-001, US-002          |
| 4    | US-004 | Asignación de Usuario a Obra                  | Acceso y Seguridad  | 3      | S       | Must    | US-003, US-005          |
| 5    | US-014 | Tablero de Usuarios con Filtros Avanzados     | Acceso y Seguridad  | 3      | S       | Must    | US-003, US-004          |
| 6    | US-005 | Gestión de Obras                              | Obras               | 8      | L       | Must    | US-001                  |
| 7    | US-006 | Gestión de Sub-Etapas                         | Obras               | 3      | S       | Must    | US-005                  |
| 8    | US-008 | Carga Semanal de Archivo XML                  | Cargas              | 13     | XL      | Must    | US-004, US-005          |
| 9    | US-010 | Control de Avances — Vistas Gantt y PTS       | Control de Avances  | 13     | XL      | Must    | US-008                  |
| 10   | US-012 | Cierre de Control por Controlador             | Control de Avances  | 5      | M       | Must    | US-010                  |
| 11   | US-013 | Cierre de Validación (Manual y Automático)    | Control de Avances  | 8      | L       | Must    | US-012                  |
| 12   | US-007 | Gestión de Tipos y Estados de Obra            | Obras               | 2      | XS      | Should  | US-001                  |
| 13   | US-009 | Habilitar Vista Física                        | Cargas              | 2      | XS      | Should  | US-008                  |
| 14   | US-011 | Control de Avances — Vista Física             | Control de Avances  | 5      | M       | Should  | US-009, US-010          |

---

## Plan de Lanzamiento

### Fase 1 — MVP (Sprints 1–6)
**Historias**: US-001, US-002, US-003, US-004, US-014, US-005, US-006, US-008, US-010, US-012, US-013  
**Puntos totales**: 66  
**Sprints estimados**: 6 sprints de ~11 puntos promedio  
**Resultado**: Sistema operativo completo para el ciclo semanal de control de avance físico.

### Fase 2 — v1.1 (Sprints 7–8)
**Historias**: US-007, US-009, US-011  
**Puntos totales**: 9  
**Resultado**: Vista Física habilitada, gestión de catálogos de tipos y estados de obra.

---

## Mapa de Dependencias

```
US-001 (Login)
  └── US-002 (Perfiles/Permisos)
        └── US-003 (Usuarios)
              └── US-004 (Asignación Usuario-Obra)
                    ├── US-014 (Tablero de Usuarios)
                    └── US-008 (Carga XML)
                          └── US-010 (Vistas Gantt/PTS)
                                └── US-012 (Cierre Control)
                                      └── US-013 (Cierre Validación)
  └── US-005 (Obras)
        ├── US-006 (Sub-Etapas)
        └── US-007 (Tipos/Estados)
  └── US-009 (Habilitar Vista Física)
        └── US-011 (Vista Física) ← también depende de US-010
```

---

## Riesgos y Supuestos

| Riesgo | Impacto | Mitigación |
|--------|---------|------------|
| US-008 y US-010 son las historias más complejas (13 pts c/u). Si el formato XML varía entre proyectos, el parser puede requerir más esfuerzo. | Alto | Definir y documentar el esquema XSD antes de iniciar US-008. |
| La regla de ventana horaria (jueves-jueves 08:00) requiere manejo de zona horaria (America/Santiago). | Medio | Implementar validación de timezone desde el inicio en US-008. |
| El cierre automático de validación (US-013) agrega complejidad al flujo de cierre de control (US-012). | Medio | Implementar US-013 como extensión de US-012, con cobertura de tests end-to-end. |
| La restricción de un solo Validador activo por obra requiere lógica de unicidad en US-004. | Medio | Agregar constraint de base de datos y validación en capa de servicio. |

