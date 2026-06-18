# Product Backlog: Refactor y Mejoras — Tree-Table de Actividades

## Metadatos
- **PRD Fuente**: `ai-specs/specs/PRD.md`
- **Generado**: 2026-06-16
- **Método de Priorización**: RICE Scoring + MoSCoW
- **Método de Estimación**: Fibonacci Story Points + Planning Poker + T-Shirt Sizing
- **Total de Historias**: 12
- **Total de Story Points**: 55

## Alcance MVP
El MVP incluye todas las historias clasificadas como **Must Have**: la corrección del bug de deep linking (US-012), el control global de expansión/colapso (US-006), la interfaz de filtros por columna (US-001) y su algoritmo recursivo de filtrado (US-002). Estas cuatro historias resuelven los problemas críticos de navegación y filtrado que impactan directamente la productividad diaria de todos los usuarios del Tree-Table.

## Resumen de Épicas

### Épica 1: Optimización y Control Jerárquico en Tree-Table de Actividades
- **Descripción**: Mejoras al componente Tree-Table incluyendo filtros avanzados por columna, visualización de estados de restricción, ajustes estéticos y control global de expansión.
- **Objetivo de Negocio**: Soporta REQ-01, REQ-02, REQ-03, REQ-04 del PRD.
- **Historias**: US-001, US-002, US-003, US-004, US-005, US-006
- **Puntos Totales**: 26
- **Prioridad**: Must / Should

### Épica 2: Gestión y Flexibilidad de Restricciones
- **Descripción**: Nuevas funcionalidades para gestionar restricciones: cierre directo, búsqueda predictiva de responsables y flexibilización de reglas de fechas.
- **Objetivo de Negocio**: Soporta REQ-05, REQ-06, REQ-07 del PRD.
- **Historias**: US-007, US-008, US-009
- **Puntos Totales**: 13
- **Prioridad**: Should / Could

### Épica 3: ProcesMassive & Data Sync
- **Descripción**: Carga masiva de tareas y restricciones mediante archivos Excel con validación de integridad del árbol.
- **Objetivo de Negocio**: Soporta REQ-08 del PRD.
- **Historias**: US-010, US-011
- **Puntos Totales**: 13
- **Prioridad**: Could

### Épica 4: Soporte y Estabilidad
- **Descripción**: Corrección de errores en enlaces profundos de notificaciones por correo electrónico.
- **Objetivo de Negocio**: Soporta REQ-09 del PRD.
- **Historias**: US-012
- **Puntos Totales**: 3
- **Prioridad**: Must

## Backlog Priorizado

| Rango | ID     | Título                                              | Épica | Puntos | T-Shirt | MoSCoW     | Dependencias |
|-------|--------|------------------------------------------------------|-------|--------|---------|------------|--------------|
| 1     | US-012 | Corrección de deep linking en notificaciones         | 4     | 3      | S       | Must       | —            | 
| 2     | US-006 | Control global de expansión y colapso                | 1     | 3      | S       | Must       | —            |  8 hh C
| 3     | US-001 | Interfaz de filtros por columna en Tree-Table        | 1     | 5      | M       | Must       | —            |  US-002
| 4     | US-002 | Algoritmo recursivo de filtrado con ancestros        | 1     | 8      | L       | Must       | US-001       |  US-001 + US-002 -> 18 hh C / B
| 5     | US-009 | Flexibilización de selección de fechas               | 2     | 3      | S       | Should     | —            |  4 HH  
| 6     | US-003 | Badge de estado de restricción en nodos hoja         | 1     | 3      | S       | Should     | —            |  10 HH C / B
| 7     | US-005 | Ajuste de alineación de texto en columna Nombre      | 1     | 2      | XS      | Should     | —            |  0 HH
| 8     | US-007 | Cierre directo de restricciones                      | 2     | 5      | M       | Could      | —            |  8 HH  C / B 
| 9    | US-008 | Búsqueda predictiva de responsables                  | 2     | 5      | M       | Could      | —             |  0 HH
| 10    | US-010 | Carga de Excel para actualización masiva             | 3     | 8      | L       | Could      | —            |  50 HH  
| 11     | US-011 | Validación de integridad jerárquica post-carga       | 3     | 5      | M       | Could      | US-010      |  2 HH
  TOTAL :                                                                                                                                100 HH 

## Plan de Liberación

### Fase 1 — MVP
- **Historias**: US-012, US-006, US-001, US-002
- **Puntos estimados**: 19
- **Sprints objetivo**: 2 sprints (Sprint 1: US-012 + US-006 + US-001; Sprint 2: US-002 + validación)
- **Justificación**: Se corrige el bug crítico de deep linking, se habilita navegación con expandir/colapsar todo, y se implementa el filtrado por columna con su lógica recursiva. Sin estas funcionalidades la tabla no es completamente usable.

### Fase 2 — v1.1 (Should Have)
- **Historias**: US-009, US-003, US-005
- **Puntos estimados**: 8
- **Sprints objetivo**: 1 sprint
- **Justificación**: Mejoras importantes de UX: flexibilidad en fechas de restricciones, visualización de estados y ajuste estético de la columna principal.

### Fase 3 — v1.2 (Could Have)
- **Historias**: US-007, US-011, US-008, US-004, US-010
- **Puntos estimados**: 28
- **Sprints objetivo**: 2-3 sprints
- **Justificación**: Funcionalidades de productividad y carga masiva. Se entregan cuando el equipo haya estabilizado las mejoras core.

## Mapa de Dependencias
- US-002 → depende de US-001 (el algoritmo necesita la interfaz de filtros)
- US-004 → depende de US-003 (el resumen padre necesita el badge individual)
- US-011 → depende de US-010 (la validación requiere la carga implementada)

## Riesgos y Supuestos
- **Riesgo**: El algoritmo recursivo de filtrado (US-002) puede tener impacto en rendimiento con árboles de gran profundidad (>10 niveles). Se recomienda evaluar memoización o virtualización.
- **Riesgo**: La carga masiva de Excel (US-010) requiere coordinación con el equipo de backend para el parser y las validaciones de integridad referencial.
- **Supuesto**: El componente Tree-Table existente utiliza un estado plano con referencia a padres mediante `parentId`, no un árbol anidado recursivo. Esto puede afectar la implementación del filtrado.
- **Supuesto**: El framework frontend detecta cambios de forma eficiente al mutar la propiedad `expanded` en los nodos. Si no es así, US-006 requerirá optimización adicional.
- **Riesgo**: La flexibilización de fechas (US-009) puede tener implicaciones en reportes o validaciones aguas abajo. Requiere revisión con el equipo de negocio.
