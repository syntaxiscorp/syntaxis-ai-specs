---
documento: Requerimientos de Refactor y Mejoras
origen: Feedback de Producción (Ajuste Técnico Tree-Table)
estado: Pendiente de Estimación
tags:
  - smat/backlog
  - agile/user-stories
  - frontend/treetable
---

# 📋 Especificación de Requerimientos (Especialización para Tree-Table)

> [!important] Nota de Arquitectura Frontend
> La "Tabla de Actividades" está implementada como un componente **Tree-Table** (Estructura jerárquica de nodos). Toda manipulación de datos, filtrado y renderizado debe considerar la recursividad de los nodos (`children`) para no romper la integridad del árbol.

---

## 🏗️ Épica 1: Optimización y Control Jerárquico en "Tree-Table de Actividades"

### REQ-01: Filtros Avanzados por Columna en Estructura de Árbol
* **Tipo:** Mejora de Funcionalidad Core
* **Descripción:** Incorporar campos de búsqueda por columna en el encabezado del Tree-Table. 
* **Complejidad de Estimación para la IA:** El algoritmo de filtrado debe ser recursivo. Si un nodo hijo coincide con el filtro, toda su rama ancestral (nodos padres) debe permanecer visible para mantener el contexto del árbol, ocultando únicamente los nodos hermanos que no apliquen.

### REQ-02: Visualización de Estados de Restricción en Nodos
* **Tipo:** UI/UX & Data Binding Jerárquico
* **Descripción:** Homologar la visualización del estado de las restricciones dentro del Tree-Table para que se renderice con la misma estética que en las tareas tradicionales.
* **Complejidad de Estimación para la IA:** Mapear el badge de estado directamente en las filas del Tree-Table evaluando si la restricción afecta a un nodo hoja (actividad final) o si debe resumir de forma visual el estado de sus nodos dependientes.

### REQ-03: Ajuste Estético de la Columna "Nombre" (Respetando Indentación)
* **Tipo:** Refactor UI CSS Avanzado
* **Descripción:** Mejorar la justificación/alineación del texto en la columna "Nombre".
* **Complejidad de Estimación para la IA:** Al ser un Tree-Table, la columna "Nombre" contiene el botón para expandir/colapsar (chevron) y el margen de profundidad de los nodos hijos (`padding-left` dinámico por nivel). La alineación del texto debe respetar estrictamente estos elementos nativos para no romper la sangría visual de la jerarquía de la obra.

### REQ-04: Control Global de Expansión (Expandir/Reducir Todo)
* **Tipo:** Feature de Navegación de Árbol
* **Descripción:** Implementar una acción masiva para `Expandir Todo` y `Colapsar Todo` los nodos del Tree-Table mediante una botonera superior.
* **Complejidad de Estimación para la IA:** Requiere un método en el frontend que recorra recursivamente el arreglo de nodos en memoria y altere la propiedad de estado de expansión (ej. `node.expanded = true/false`), gatillando la detección de cambios del framework de forma eficiente sin congelar la UI.

---

## 🛡️ Épica 2: Gestión y Flexibilidad de Restricciones

### REQ-05: Cierre Directo de Restricciones
* **Tipo:** Nueva Funcionalidad (Workflow)
* **Descripción:** Permitir cerrar restricciones directamente desde el apartado de "Restricciones".

### REQ-06: Búsqueda Predictiva de Responsables
* **Tipo:** Optimización UI/UX (Autocomplete)
* **Descripción:** Implementar un buscador predictivo reactivo para la asignación de responsables en restricciones.

### REQ-07: Flexibilización de Reglas de Negocio en Fechas
* **Tipo:** Modificación de Lógica de Negocio
* **Descripción:** Permitir la selección de fechas anteriores a la "fecha necesaria" al momento de generar una restricción.

---

## 🚀 Épica 3: ProcesMassive & Data Sync

### REQ-08: Carga Masiva y Mapeo de Tareas/Restricciones en Árbol
* **Tipo:** Core Performance (Bulk Processing)
* **Descripción:** Habilitar la carga de archivos Excel (`.xlsx`) para la actualización masiva.
* **Complejidad de Estimación para la IA:** El parser del backend debe validar que las actualizaciones del Excel respeten las relaciones de herencia y llaves foráneas de la estructura del árbol existente, asegurando que no queden nodos huérfanos tras procesar el archivo.

---

## 🐛 Épica 4: Soporte y Estabilidad

### REQ-09: Corrección de Deep Linking (Notificaciones por Correo)
* **Tipo:** Bug Fix (Ruteo / Autenticación)
* **Descripción:** Resolver el error al acceder a la plataforma desde los links de correos electrónicos.