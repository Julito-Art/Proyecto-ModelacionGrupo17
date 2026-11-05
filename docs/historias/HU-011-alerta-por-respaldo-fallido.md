# HU-011 – Recibir una alerta cuando un respaldo falla

| | |
|---|---|
| **Prioridad (MoSCoW)** | Must |
| **Estimación** | 5 story points |
| **Sprint** | 3 |
| **Épica / Feature** | E2 Monitoreo y alertas / F2.2 Alertas y notificaciones |
| **Depende de** | HU-005 |
| **Origen en OAS** | Relevamiento (el incidente sufrido por la empresa); Diagnóstico. |
| **Documentos relacionados** | [06 – Clases](../06-diseno-de-clases.md), [07 – Estados y secuencia](../07-estados-y-secuencia.md), [Casos de prueba](../testing/casos-HU-011.md), [10 – Prototipo y heurísticas](../10-prototipo-y-heuristicas.md) |

## Historia

Como **Responsable de Respaldos** quiero recibir una alerta por correo y en el panel cuando un respaldo queda FALLIDA tras agotar los reintentos para actuar antes de que se acumule la pérdida de datos.

## Contexto y reglas de negocio

- Cuando un respaldo agota sus reintentos queda FALLIDA y el servidor pasa a estar sin protección. La alerta existe para que el Responsable lo sepa antes de que se acumule la pérdida de datos; es lo que le faltó a la empresa en el incidente que relata el trabajo de OAS.
- La alerta es de tipo FALLA y severidad CRITICA. Se muestra siempre en el panel y, además, se envía por correo (SM-07); si el correo falla, la alerta no se pierde.
- Avisar a Dirección de las alertas críticas es la historia HU-013 (Sprint 4): en esta historia el destinatario es el Responsable de Respaldos. El diagrama de secuencia del documento 07 muestra el comportamiento completo, con HU-013 ya implementada.

**Reglas que aplican** (documento maestro):

- **RN-06:** Agotados los reintentos, la tarea queda FALLIDA y se genera una alerta CRITICA.
- **RN-08:** Hay como máximo una alerta ABIERTA por servidor y tipo; pasa a CERRADA cuando un respaldo posterior del servidor resulta exitoso.
- **RN-09:** Las alertas CRITICA van al Responsable de Respaldos y a Dirección (Dirección se suma en HU-013).
- **RN-13:** La generación de una alerta es una acción relevante y se audita.

## Criterios de aceptación

**CA-1 – Alerta crítica tras agotar los reintentos** · _camino normal_

- **Dado** una tarea de respaldo de TANGO que quedó FALLIDA tras agotar los 3 reintentos, con el motivo «Acronis no respondió: tiempo de espera agotado»
- **Cuando** el planificador procesa su resultado
- **Entonces** se crea una alerta de tipo FALLA y severidad CRITICA, en estado ABIERTA, con el servidor TANGO, la hora y el motivo; el Responsable la recibe por correo en menos de 1 minuto y aparece en el panel

**CA-2 – Sin alerta si un reintento tiene éxito** · _camino normal_

- **Dado** un respaldo de STRUMIS que falla en el primer intento y tiene éxito en el primer reintento
- **Cuando** la tarea termina
- **Entonces** no se genera ninguna alerta

**CA-3 – El correo falla: la alerta no se pierde** · _caso de error_

- **Dado** que el servicio de correo no responde
- **Cuando** se genera la alerta de CA-1
- **Entonces** la alerta queda igualmente ABIERTA y visible en el panel, con el aviso «No se pudo enviar el correo»; la alerta queda marcada con `correoEnviado` = falso y con el motivo del fallo del envío guardado en `motivoFalloCorreo`

**CA-4 – Cierre de la alerta** · _camino normal_

- **Dado** una alerta ABIERTA de TANGO
- **Cuando** el siguiente respaldo de TANGO resulta exitoso
- **Entonces** la alerta pasa a CERRADA y deja de figurar entre las abiertas

**CA-5 – No se duplican las alertas** · _caso de error_

- **Dado** una alerta de tipo FALLA de TANGO que ya está ABIERTA
- **Cuando** otro respaldo de TANGO vuelve a quedar FALLIDA
- **Entonces** no se crea otra alerta: sigue habiendo una sola ABIERTA para ese servidor y tipo, que actualiza su motivo y su hora con los de la nueva falla; no se reenvía el correo ni se registra otra ALERTA_GENERADA

**CA-6 – La alerta queda auditada** · _camino normal_

- **Dado** una alerta generada
- **Cuando** se crea
- **Entonces** se registra ALERTA_GENERADA en la auditoría con el usuario «SGR», la fecha y hora, el servidor y la severidad

**CA-7 – Las alertas son por servidor** · _camino normal_

- **Dado** una alerta de tipo FALLA de TANGO ABIERTA
- **Cuando** un respaldo de STRUMIS queda FALLIDA
- **Entonces** se crea una segunda alerta, de STRUMIS, y quedan dos ABIERTAS (una por servidor); la de TANGO no cambia

## Qué queda fuera de esta historia

- Notificar a Dirección (HU-013).
- Alertas por atraso de respaldos, ADVERTENCIA o CRITICA por tiempo (HU-012).
- Los reintentos del respaldo (HU-005) y la verificación de integridad (HU-008).
- Reenviar el correo a mano o reintentar su envío automático.

## Notas de diseño

**Clases del diseño (documento 06) que toca:**

- `GestorDeAlertas`: `alertarFalla(tarea, ahora)`, `cerrarAlertas(servidor, ahora)` y `alertasAbiertas()`; busca la alerta abierta con `IRepositorioDeAlertas.abierta(servidor, tipo)` y la guarda con `guardar(alerta)`.
- `Alerta`: constructor, `actualizar(mensaje, ahora)` (CA-5), `cerrar(ahora)`, `estado()`, `tipo()`, `servidor()`, `mensaje()` y `destinatarios()`; para CA-3, el atributo `correoEnviado` con `marcarCorreoEnviado()`, `marcarCorreoFallido(motivo)` y `correoEnviado()`; enumeraciones `TipoDeAlerta`, `Severidad` y `EstadoAlerta`.
- `INotificador.notificar(alerta, roles)` y `NotificadorPorCorreo` (puerto y adaptador de correo). Si el envío falla, `notificar` lanza `NotificacionException`; `GestorDeAlertas.alertarFalla` la captura, llama a `Alerta.marcarCorreoFallido(motivo)` y deja la alerta ABIERTA y visible en el panel.
- `TareaDeRespaldo` (`estado()`, `motivoFallo()`), `PlanificadorDeRespaldos.procesarResultado` y `ServicioDeAuditoria.registrar`.

Esta historia usa el **Singleton**: es el `PlanificadorDeRespaldos` (la única instancia) quien llama a `GestorDeAlertas.alertarFalla` al procesar el resultado de la tarea.

**Capturas finales de las pantallas** (ya corregidas después de la evaluación heurística, ver el [documento 10](../10-prototipo-y-heuristicas.md#hu-011)):

- [Pantalla 1 – Panel de alertas](../../prototipo/HU-011-p1-panel-alertas.png)
- [Pantalla 2 – Correo de alerta](../../prototipo/HU-011-p2-correo.png)
- [Pantalla 3 – Alerta cuando el correo falla](../../prototipo/HU-011-p3-correo-fallido.png)

## Datos de prueba

| Criterio | Datos de prueba |
|---|---|
| CA-1 | Reloj simulado 26/10/2026. Tarea de TANGO pasa a FALLIDA a las 14:07:00. Alerta creada a las 14:07:02. Correo enviado a las 14:07:30 (28 segundos: cumple). Bordes del «menos de 1 minuto»: envío a los 59 segundos cumple; a los 60 y a los 61, no. Destinatario: resp.prueba@prometva.test. Asunto esperado: «[SGR] CRITICA: el respaldo de TANGO falló». |
| CA-2 | Acronis simulado falla en el intento 1 de STRUMIS y responde «exitoso» e «íntegro» en el reintento 1. Esperado: 0 alertas. |
| CA-3 | Servidor de correo de prueba apagado (conexión rechazada). Esperado: 1 alerta ABIERTA; aviso «No se pudo enviar el correo» en el panel; `correoEnviado` = falso y `motivoFalloCorreo` = «Conexión rechazada por el servidor de correo». |
| CA-4 | Alerta ABIERTA de TANGO; respaldo exitoso de TANGO a las 14:12. Esperado: estado CERRADA y fecha de cierre 14:12. |
| CA-5 | Alerta FALLA de TANGO ABIERTA desde las 14:07 con el motivo «Acronis no respondió: tiempo de espera agotado»; segunda tarea de TANGO FALLIDA a las 14:20 con «Acronis: error de autenticación». Esperado: 1 alerta ABIERTA (no 2), con motivo «Acronis: error de autenticación» y actualizada 14:20; 0 correos nuevos y 0 registros ALERTA_GENERADA nuevos. |
| CA-6 | Alerta de CA-1. Registro esperado: usuario «SGR», acción ALERTA_GENERADA, objetivo TANGO, resultado «CRITICA», fecha y hora 26/10/2026 14:07:02. |
| CA-7 | Alerta FALLA de TANGO ABIERTA; tarea de STRUMIS FALLIDA a las 14:25. Esperado: 2 alertas ABIERTAS (TANGO y STRUMIS). |

## Decisiones de implementación

- **Dónde se ve la alerta en el Sprint 3:** esta historia incluye el panel de alertas mínimo (la lista de alertas abiertas, pantalla 1 del documento 10); el panel completo de estado es la HU-009. Por eso HU-011 no depende de HU-009.
- **Dónde se guarda el motivo del fallo del correo:** en la propia alerta (`motivoFalloCorreo`), junto a `correoEnviado`.
- **Correos por rol:** `NotificadorPorCorreo` pide los usuarios activos de cada rol a `IRepositorioDeUsuarios.usuariosActivosConRol(rol)` y envía a sus correos.
- **Falla repetida con una alerta ya abierta (CA-5):** no se crea otra; `Alerta.actualizar(mensaje, ahora)` cambia el motivo y la hora, no se vuelve a enviar el correo y no se registra otra ALERTA_GENERADA (la nueva falla ya queda en RESPALDO_FINALIZADO).
- **Sin Responsable activo:** si `usuariosActivosConRol` no devuelve a nadie, el notificador lanza `NotificacionException` con «No hay usuarios activos con el rol Responsable de Respaldos»; la alerta queda como en CA-3.
- **Panel de alertas:** lo ven el Responsable de Respaldos y Dirección (esta en solo lectura). Las alertas abiertas se ordenan por severidad (CRITICA primero) y, dentro de cada severidad, de la más reciente a la más antigua. Las columnas son siempre las mismas: Severidad, Tipo, Servidor, Desde, Estado y Motivo. Sin alertas abiertas se muestra «No hay alertas abiertas».
- **Cierre de alertas:** un respaldo exitoso del servidor cierra sus alertas abiertas de FALLA y de ATRASO (`cerrarAlertas`).
- **«Menos de 1 minuto» (CA-1):** se mide desde que la alerta se crea hasta que se envía el correo.

## Dependencias, estimación y Definition of Ready

- **Depende de:** HU-005 (reintentos: de ahí sale el estado FALLIDA).
- **Estimación:** 5 SP, Sprint 3.

**Chequeo de Definition of Ready:**

- [x] Tiene rol, acción y beneficio, idénticos a los del backlog ([documento 01](../01-vision-y-backlog.md)).
- [x] Tiene 7 criterios Dado/Cuando/Entonces y 2 son casos de error.
- [x] Tiene estimación (5 SP) y prioridad (Must).
- [x] Sus dependencias están planificadas en sprints anteriores o en el mismo sprint (documento 02).
- [x] Indica qué queda fuera y los datos de prueba de cada criterio.
- [x] Revisada por los cuatro integrantes del grupo.
