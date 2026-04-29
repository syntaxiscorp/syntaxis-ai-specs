

Como Validador, quiero revisar los avances registrados por todos los controladores y realizar el cierre de validación semanal, para certificar oficialmente el avance de la obra y habilitar la descarga de los archivos.

Story Points: 8 | T-Shirt: L | MoSCoW: Must Have



Criterios de Aceptación:
1. Panel muestra lista de controladores con estado (Abierto/Cerrado) y contador de pendientes.
2. Cierre manual exitoso → VALIDACION_SEMANA APROBADO + VALIDACION_LOTE por cada lote + archivos descargables.
3. Controlador intenta cerrar validación → HTTP 403.
4. Descarga post-validación → presigned URL con TTL 15 min.
5. Descarga sin validación cerrada → HTTP 403.

Dependencias: Bloqueado por US-012.