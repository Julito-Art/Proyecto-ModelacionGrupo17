# Casos de prueba de HU-011 – Recibir una alerta cuando un respaldo falla

| | |
|---|---|
| **Historia** | [HU-011](../historias/HU-011-alerta-por-respaldo-fallido.md) |
| **Plan de testing** | [plan-de-testing.md](plan-de-testing.md) |
| **Estado** | Diseñados (el SGR todavía no está implementado: no hay resultados de ejecución) |

## Casos

| ID | Criterio | Nivel | Tipo | Precondiciones | Pasos | Datos de entrada | Resultado esperado | RNF |
|---|---|---|---|---|---|---|---|---|
| CP-011-01 | CA-1 | Integración | normal | Tarea de TANGO FALLIDA a las 14:07:00 con el motivo indicado; correo simulado disponible. | Procesar el resultado de la tarea. | Datos de CA-1. | Alerta FALLA, CRITICA y ABIERTA con servidor TANGO, hora y motivo; aparece en el panel y se notifica al Responsable. | — |
| CP-011-02 | CA-1 | Integración | límite | Igual que CP-011-01. | Medir el tiempo entre la creación de la alerta y el envío del correo. | Envío a los 59, a los 60 y a los 61 segundos. | A los 59 segundos cumple («en menos de 1 minuto»); a los 60 y a los 61, no. | — |
| CP-011-03 | CA-1 | E2E | normal | Servidor SMTP de prueba; sesión de Responsable. | Provocar el fallo de TANGO y abrir el correo recibido y el panel. | Destinatario resp.prueba@prometva.test. | Correo con asunto «[SGR] CRITICA: el respaldo de TANGO falló», servidor, hora y motivo; la alerta figura ABIERTA en el panel con su motivo. | — |
| CP-011-04 | CA-2 | Integración | negativo | Acronis simulado falla en el intento 1 de STRUMIS y funciona en el reintento 1. | Ejecutar el ciclo. | Intento 1 con error; reintento 1 exitoso. | Tarea COMPLETADA y 0 alertas. | — |
| CP-011-05 | CA-3 | Integración | negativo | `INotificador` simulado que lanza error de conexión. | Generar la alerta de CP-011-01. | Servidor de correo apagado; motivo «Conexión rechazada por el servidor de correo». | `notificar` lanza `NotificacionException`; la alerta queda ABIERTA y visible en el panel con «No se pudo enviar el correo», con `correoEnviado()` = falso y `motivoFalloCorreo()` = «Conexión rechazada por el servidor de correo»; no se pierde. | — |
| CP-011-06 | CA-4 | Integración | normal | Alerta ABIERTA de TANGO. | Completar un respaldo exitoso de TANGO a las 14:12 y llamar a `GestorDeAlertas.cerrarAlertas`. | Respaldo exitoso a las 14:12. | La alerta pasa a CERRADA con fecha de cierre 14:12 y no figura entre las abiertas. | — |
| CP-011-07 | CA-5 | Integración | negativo | Alerta FALLA de TANGO ABIERTA desde las 14:07 con motivo «Acronis no respondió: tiempo de espera agotado»; correo simulado que cuenta los envíos. | Hacer que otra tarea de TANGO quede FALLIDA a las 14:20 con «Acronis: error de autenticación». | Segunda falla de TANGO. | Sigue habiendo una sola alerta ABIERTA para TANGO y tipo FALLA, con motivo «Acronis: error de autenticación» y actualizada a las 14:20; 0 correos nuevos y 0 registros ALERTA_GENERADA nuevos. | — |
| CP-011-08 | CA-5 | Unitario | límite | `GestorDeAlertas` con un repositorio de alertas simulado y 0 alertas abiertas. | Llamar a `alertarFalla` dos veces seguidas para el mismo servidor. | Dos fallas consecutivas. | Después de la primera hay 1 abierta; después de la segunda siguen siendo 1 (el máximo), y `Alerta.actualizar` se llamó una vez. | — |
| CP-011-09 | CA-6 | Integración | normal | Alerta generada en CP-011-01. | Consultar la auditoría. | — | Registro ALERTA_GENERADA con usuario «SGR», objetivo TANGO, resultado «CRITICA» y fecha y hora 26/10/2026 14:07:02. | RNF-06 |
| CP-011-10 | CA-7 | Integración | normal | Alerta FALLA de TANGO ABIERTA. | Hacer que una tarea de STRUMIS quede FALLIDA a las 14:25. | Falla de STRUMIS. | Quedan 2 alertas ABIERTAS (TANGO y STRUMIS); la de TANGO no cambia. | — |
| CP-011-11 | CA-1 | Integración | normal | Reloj simulado; correo simulado que demora 28 segundos. | Generar la alerta de TANGO a las 14:07:02 y medir el envío. | Envío a las 14:07:30 (dato de CA-1 de la historia). | El correo sale a los 28 segundos de creada la alerta: cumple «en menos de 1 minuto». | — |
| CP-011-12 | CA-3 | E2E | negativo | Servidor SMTP de prueba apagado; sesión de Responsable. | Provocar la falla de TANGO y abrir el panel de alertas. | — | El panel muestra el aviso «No se pudo enviar el correo de esta alerta» y la alerta ABIERTA con las mismas columnas que la pantalla 1 (Severidad, Tipo, Servidor, Desde, Estado y Motivo). | — |

## Matriz de trazabilidad criterio ↔ caso de prueba

| Criterio de aceptación | Casos de prueba |
|---|---|
| CA-1 | CP-011-01, CP-011-02, CP-011-03, CP-011-11 |
| CA-2 | CP-011-04 |
| CA-3 | CP-011-05, CP-011-12 |
| CA-4 | CP-011-06 |
| CA-5 | CP-011-07, CP-011-08 |
| CA-6 | CP-011-09 |
| CA-7 | CP-011-10 |

## Cobertura

- Criterios de aceptación: 7; criterios con al menos un caso que verifica todo su «Entonces»: 7.
- Casos negativos (error o dato inválido): CP-011-04, CP-011-05, CP-011-07, CP-011-12.
- Casos de valor límite: CP-011-02, CP-011-08.
- Casos ligados a un RNF: CP-011-09 (RNF-06).
