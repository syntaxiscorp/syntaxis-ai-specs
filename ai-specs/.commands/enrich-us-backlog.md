# AGENT: Salfa SMAT - Task Strategist (Frontend/Backend/Testing)

## 🎯 PROPÓSITO
Analizar las Historias de Usuario (HUs) en el directorio `backlog/` (formato `us-xxx.md`) y transformarlas en un plan de trabajo claro dividido en tres pilares: Frontend, Backend y uiere Testing. La intencion es mantener la información que actualmente tiene la HU sin embargo se requiere agregar las tareas / subtareas correspondiente a las indicaciones entregadas

## 🛠 CONTEXTO TÉCNICO DE REFERENCIA
- **Backend:** .NET 9, Clean Architecture (Domain, Application, Infra), EF Core 9, Autommaper.
- **Frontend:** [Angular], Consumo de Minimal APIs.
- **Calidad:** Result Pattern, Serilog, xUnit, Testcontainers.

---

## 📋 INSTRUCCIONES DE ENRIQUECIMIENTO

Por cada archivo `us-xxx.md` ubicdo en el directorio `.backlog/` , el agente debe generar la sección **🚀 [Enhanced - Task Plan]** siguiendo este desglose:

### 1. 🖥️ FRONTEND (Interfaz y Experiencia)
Define las tareas necesarias para la UI. Debe incluir:
- Diseño/Modificación de componentes.
- Manejo de estado y validaciones de formulario.
- Integración con los endpoints del Backend.

### 2. ⚙️ BACKEND (Lógica y Datos)
Define las tareas de servidor bajo estándares de Clean Architecture. Debe incluir:
- **Lógica:** Entidades de dominio y Commands/Queries de MediatR.
- **Datos:** Configuraciones de EF Core (Fluent API) y Repositorios.
- **API:** Definición de Endpoints y Middleware (Logs/Seguridad).

### 3. 🧪 TESTING & QA (Calidad)
Define los criterios para asegurar que nada falle. Debe incluir:
- Pruebas Unitarias de lógica crítica.
- Pruebas de Integración (Base de datos/API).
- Pruebas Manuales o de Usuario (Criterios de Aceptación).

---

## 📤 FORMATO DE SALIDA REQUERIDO

---
status: "Pending Validation"
---

# [TÍTULO DE LA HU]

## 📜 [Original]
[Contenido original del archivo]

---

## 🚀 [Enhanced - Task Plan]

### 📖 Resumen de la Solución
[Breve explicación técnica de cómo se abordará el requerimiento]

### 🛠️ Lista de Tareas y Subtareas

#### 🟦 BACKEND
- [ ] **Tarea: Implementar Lógica y Persistencia**
  - Subtarea: Crear Entidades y lógica de Dominio.
  - Subtarea: Configurar persistencia en BD y Repositorio.
  - Subtarea: Crear Command/Query y Handler (Application).
- [ ] **Tarea: Exposición de API**
  - Subtarea: Crear Endpoint y DTOs de respuesta.
  - Subtarea: Implementar Logs estructurados con Serilog.

#### 🟨 FRONTEND
- [ ] **Tarea: Desarrollo de UI**
  - Subtarea: Crear/Actualizar componentes de vista.
  - Subtarea: Implementar lógica de validación en cliente.
- [ ] **Tarea: Integración**
  - Subtarea: Configurar servicio de consumo de API (HttpClient/Axios).
  - Subtarea: Manejo de estados de carga y errores del Result Pattern.

#### 🟩 TESTING & QA
- [ ] **Tarea: Pruebas Automatizadas**
  - Subtarea: Tests Unitarios para reglas de negocio.
  - Subtarea: Tests de Integración de API con BD real.
- [ ] **Tarea: Validación de Usuario**
  - Subtarea: Verificar cumplimiento de Criterios de Aceptación (DoD).

---
## 📝 NOTAS DE ARQUITECTURA
[Dudas, dependencias o sugerencias técnicas del agente]