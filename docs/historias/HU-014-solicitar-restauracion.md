# HU-014 – Solicitar una restauración

| | |
|---|---|
| **Prioridad (MoSCoW)** | Must |
| **Estimación** | 5 story points |
| **Sprint** | 4 |
| **Épica / Feature** | E3 Restauración de datos / F3.1 Solicitud y autorización de restauración |
| **Depende de** | HU-008 |
| **Origen en OAS** | Relevamiento (hackeo y pago para recuperar los datos); Backup Empresarial (restauración granular o total). |
| **Documentos relacionados** | [06 – Clases](../06-diseno-de-clases.md), [07 – Estados y secuencia](../07-estados-y-secuencia.md), [Casos de prueba](../testing/casos-HU-014.md), [10 – Prototipo y heurísticas](../10-prototipo-y-heuristicas.md) |

## Historia

Como **Responsable de Respaldos** quiero solicitar la restauración de STRUMIS o TANGO desde un punto de restauración, de forma total o granular para recuperar los datos tras una falla o un ataque.

## Contexto y reglas de negocio

- Ante una pérdida de datos o un ataque como el que sufrió la empresa, el Responsable pide recuperar un servidor desde un punto de restauración. Pedirlo no restaura nada: la solicitud queda PENDIENTE hasta que Dirección la autorice (HU-015).
- Todo punto de restauración existe solo si el respaldo pasó la verificación de integridad (HU-008), por lo que «verificado» se cumple siempre. Lo que se filtra en esta pantalla es la vigencia: un punto vence cuando se cumple la retención de la política del servidor (por defecto 30 días; RN-12).
- La restauración puede ser TOTAL (todo el servidor) o GRANULAR (solo los elementos que se indiquen, por ejemplo ciertas tablas o carpetas).

**Reglas que aplican** (documento maestro):

- **RN-10:** Toda restauración exige solicitud del Responsable y autorización de Dirección.
- **RN-12:** Los puntos de restauración se conservan por la retención de la política del servidor vigente cuando se creó el punto (por defecto 30 días); cambiar la retención no modifica los puntos ya creados.
- **RN-13:** Solicitar una restauración es una acción relevante y se audita.
- **RN-16:** Solo el Responsable de Respaldos solicita restauraciones.
- **RN-21:** Motivo de 10 a 300 caracteres; punto obligatorio; en una restauración GRANULAR, de 1 a 20 elementos de hasta 100 caracteres cada uno, en texto libre, sin repetidos (los vacíos se descartan); en una TOTAL, los elementos se descartan.
- **RN-22:** La lista de puntos se muestra paginada (20 por página), del más reciente al más antiguo, con rango de fechas (por defecto, los últimos 7 días) y la casilla «Solo diarios»; «Desde» no puede ser posterior a «Hasta»; hay como máximo una solicitud PENDIENTE por servidor; al enviar se vuelve a comprobar la vigencia del punto.

## Criterios de aceptación

**CA-1 – Solo se ofrecen puntos vigentes** · _camino normal_

- **Dado** el servidor TANGO con tres puntos de restauración: uno de hace 2 días, uno de hace 29 días y uno de hace 31 días
- **Cuando** el Responsable abre «Nueva solicitud de restauración», elige TANGO y **amplía el rango** «Desde» a hace 35 días (por defecto el rango es de los últimos 7 días)
- **Entonces** la lista ofrece únicamente los dos puntos vigentes; el de hace 31 días no aparece aunque esté dentro del rango; con el rango por defecto solo aparece el de hace 2 días

**CA-2 – Solicitud total correcta** · _camino normal_

- **Dado** el punto de hace 2 días elegido, tipo TOTAL y el motivo «Pérdida de datos por ataque»
- **Cuando** el Responsable envía la solicitud
- **Entonces** se crea una solicitud en estado PENDIENTE con el solicitante, la fecha y el punto elegido, se muestra la confirmación con el número de solicitud y el estado PENDIENTE (pantalla 3) y se registra RESTAURACION_SOLICITADA en la auditoría con el usuario y la fecha y hora

**CA-3 – El motivo es obligatorio** · _caso de error_

- **Dado** el formulario con el motivo vacío (o solo con espacios)
- **Cuando** el Responsable envía
- **Entonces** se rechaza con «El motivo es obligatorio» junto al campo y no se crea ninguna solicitud

**CA-4 – Una restauración granular exige elementos** · _caso de error_

- **Dado** el tipo GRANULAR elegido y ningún elemento indicado
- **Cuando** el Responsable envía
- **Entonces** se rechaza con «Indicá al menos un elemento a restaurar» y no se crea ninguna solicitud

**CA-5 – Solicitud granular correcta** · _camino normal_

- **Dado** el tipo GRANULAR con los elementos «Facturas 2026» y «Clientes» y un motivo
- **Cuando** el Responsable envía
- **Entonces** se crea una solicitud PENDIENTE que conserva esos dos elementos

**CA-6 – Solo el Responsable solicita** · _caso de error_

- **Dado** un usuario con rol Dirección o Gerente de Administración autenticado
- **Cuando** intenta crear una solicitud
- **Entonces** la opción «Nueva solicitud de restauración» no aparece en su menú y, si lo intenta por otra vía, recibe «No tenés permiso para solicitar restauraciones» y no se crea nada

**CA-7 – Servidor sin puntos vigentes** · _caso de error_

- **Dado** un servidor sin ningún punto de restauración vigente
- **Cuando** el Responsable abre «Nueva solicitud de restauración» y lo elige
- **Entonces** se muestra la pantalla de estado vacío con «No hay puntos de restauración vigentes para este servidor» (pantalla 4) y no se puede enviar la solicitud

**CA-8 – Elegir un punto entre muchos** · _camino normal_

- **Dado** el servidor STRUMIS con 8.640 puntos vigentes (un incremental cada 5 minutos durante 30 días) y 2.016 de ellos en los últimos 7 días
- **Cuando** el Responsable abre «Nueva solicitud de restauración» y elige STRUMIS
- **Entonces** se listan los 20 puntos más recientes de los últimos 7 días con «Página 1 de 101»; al marcar «Solo diarios» quedan solo los diferenciales del rango (7) y la paginación pasa a «Página 1 de 1»; al cambiar el rango de fechas la lista y la cantidad de páginas se actualizan; si el rango no tiene puntos se muestra «No hay puntos de restauración en el rango elegido», y una página que no existe devuelve la lista vacía

**CA-9 – Límites del motivo y de los elementos** · _caso de error_

- **Dado** un motivo de 9 o de 301 caracteres, o una restauración GRANULAR con 21 elementos o con un elemento de 101 caracteres
- **Cuando** el Responsable envía
- **Entonces** se rechaza con «El motivo debe tener entre 10 y 300 caracteres», «Se pueden indicar hasta 20 elementos» o «Cada elemento puede tener hasta 100 caracteres», según el caso, y no se crea ninguna solicitud

**CA-10 – Una sola solicitud pendiente por servidor** · _caso de error_

- **Dado** una solicitud PENDIENTE de TANGO
- **Cuando** el Responsable envía otra solicitud de TANGO
- **Entonces** se rechaza con «Ya hay una solicitud pendiente para este servidor» y no se crea otra; una solicitud de STRUMIS sí se puede enviar

**CA-11 – Rango de fechas invertido** · _caso de error_

- **Dado** el formulario de STRUMIS
- **Cuando** el Responsable pone «Desde» 26/10/2026 y «Hasta» 19/10/2026
- **Entonces** se muestra «La fecha Desde no puede ser posterior a Hasta» junto al filtro y la lista no cambia

**CA-12 – El punto vence antes de enviar** · _caso de error_

- **Dado** un punto elegido que estaba vigente al abrir la pantalla y vence antes de que el Responsable envíe
- **Cuando** el Responsable envía la solicitud
- **Entonces** se rechaza con «El punto elegido venció. Elegí otro» y no se crea ninguna solicitud

**CA-13 – Elementos repetidos, vacíos o sobrantes** · _caso de error_

- **Dado** una restauración GRANULAR con los elementos «Clientes, clientes, Facturas 2026, , »
- **Cuando** el Responsable envía
- **Entonces** se rechaza con «Hay elementos repetidos: indicá cada uno una sola vez» (la comparación no distingue mayúsculas; los elementos vacíos entre comas se descartan sin error) y no se crea ninguna solicitud; en una restauración TOTAL con elementos escritos, la solicitud se crea y los elementos se descartan

**CA-14 – Envío sin elegir punto** · _caso de error_

- **Dado** el formulario de TANGO sin ningún punto elegido
- **Cuando** el Responsable envía
- **Entonces** se rechaza con «Elegí un punto de restauración» junto a la lista y no se crea ninguna solicitud

## Qué queda fuera de esta historia

- Autorizar o rechazar la solicitud (HU-015).
- Ejecutar la restauración y el respaldo previo a restauración (HU-016).
- Los simulacros de restauración (HU-017).
- Elegir un destino de restauración distinto del servidor de origen.

## Notas de diseño

**Clases del diseño (documento 06) que toca:**

- `ServicioDeRestauracion.solicitar(punto, tipo, elementos, motivo, solicitante, ahora)`: comprueba el rol (lanza `PermisoDenegadoException`, CA-6), llama a `SolicitudDeRestauracion.validar` y a `hayPendiente(servidor)`, y si hay errores lanza `ValidacionException` con todos los mensajes (CA-3, 4, 9, 10, 12, 13 y 14). Si no, crea la solicitud y la guarda con `IRepositorioDeSolicitudes.guardar`.
- `ServicioDeRestauracion.puntosVigentes(servidor, desde, hasta, soloDiarios, pagina, ahora)` y `contarPuntosVigentes(...)`: leen de `IRepositorioDePuntos.vigentes` y `contarVigentes`, paginados (`TAMANO_PAGINA` = 20), para CA-1, CA-7 y CA-8. Si «Desde» es posterior a «Hasta», lanzan `ValidacionException` (CA-11).
- `ServicioDeRestauracion.hayPendiente(servidor)` usa `IRepositorioDeSolicitudes.pendienteDe(servidor)` (CA-10).
- `SolicitudDeRestauracion.validar(punto, tipo, elementos, motivo, ahora)`, `separarElementos(texto)` y sus constantes `MOTIVO_MIN`, `MOTIVO_MAX`, `ELEMENTOS_MAX` y `ELEMENTO_LARGO_MAX` (CA-3, 4, 9, 12, 13 y 14).
- `SolicitudDeRestauracion` (constructor, estado inicial PENDIENTE; ver el diagrama de estados del documento 07), con `TipoDeRestauracion` (`TOTAL`, `GRANULAR`) y `elementos: List<String>`.
- `PuntoDeRestauracion.estaVigente(ahora)` (vigente si `ahora < vigenteHasta`, con `vigenteHasta = fechaHora + retención de la política` (por defecto 30 días)).
- `Usuario.tieneRol(rol)` para el control de CA-6 y `ServicioDeAuditoria.registrar` con `AccionAuditada.RESTAURACION_SOLICITADA`.

Esta historia **no** introduce un patrón nuevo; usa los puntos de restauración que genera `PlanificadorDeRespaldos` (Singleton) al completar los respaldos de HU-004. La historia de este grupo que usa patrones es HU-004.

**Capturas finales de las pantallas** (ya corregidas después de la evaluación heurística, ver el [documento 10](../10-prototipo-y-heuristicas.md#hu-014)):

- [Pantalla 1 – Nueva solicitud de restauración](../../prototipo/HU-014-p1-solicitud.png)
- [Pantalla 2 – Errores de validación](../../prototipo/HU-014-p2-errores.png)
- [Pantalla 3 – Solicitud enviada (PENDIENTE)](../../prototipo/HU-014-p3-confirmacion.png)
- [Pantalla 4 – Servidor sin puntos vigentes (estado vacío)](../../prototipo/HU-014-p4-sin-puntos.png)

## Datos de prueba

| Criterio | Datos de prueba |
|---|---|
| CA-1 | Retención de la política: 30 días (por defecto). Fecha actual simulada: 26/10/2026 23:00. Puntos de TANGO: PR-A (24/10/2026 23:00, DIFERENCIAL), PR-B (27/09/2026 23:00, INCREMENTAL) y PR-C (25/09/2026 23:00, INCREMENTAL). Rango ampliado: Desde 21/09/2026, Hasta 26/10/2026. Esperado: se ofrecen PR-A y PR-B; PR-C no. Con el rango por defecto (19/10 al 26/10): solo PR-A. Borde: un punto con `vigenteHasta` = ahora ya no es vigente. |
| CA-2 | Punto PR-A; tipo TOTAL; motivo «Pérdida de datos por ataque»; usuario resp.prueba@prometva.test. Esperado: solicitud PENDIENTE; auditoría RESTAURACION_SOLICITADA. |
| CA-3 | Motivo «» y motivo «   » (tres espacios). Esperado: rechazo en ambos casos. |
| CA-4 | Tipo GRANULAR con 0 elementos (rechazo) y con 1 elemento «Clientes» (se acepta). |
| CA-5 | Elementos «Facturas 2026» y «Clientes»; motivo «Recuperar facturación eliminada por error». |
| CA-10 | Solicitud S-12 de TANGO en PENDIENTE; nueva solicitud de TANGO con PR-A; luego una de STRUMIS. Esperado: la de TANGO se rechaza y la de STRUMIS queda PENDIENTE. |
| CA-11 | Desde 26/10/2026, Hasta 19/10/2026. |
| CA-12 | Punto con `vigenteHasta` 26/10/2026 23:00:00; pantalla abierta a las 22:58; envío a las 23:00:05. |
| CA-13 | GRANULAR con «Clientes, clientes, Facturas 2026, , » (rechazo por repetidos); GRANULAR con «Clientes, , Facturas 2026» (se acepta con 2 elementos); TOTAL con «Clientes» (se crea con 0 elementos). |
| CA-14 | TANGO, ningún punto elegido, tipo TOTAL, motivo «Pérdida de datos por ataque». |
| CA-6 | direccion.prueba@prometva.test y gerente.prueba@prometva.test. |
| CA-7 | Servidor STRUMIS sin puntos vigentes (todos vencidos según la retención por defecto de 30 días). |
| CA-8 | STRUMIS con 8.640 puntos vigentes (2.016 en los últimos 7 días, 7 de ellos diferenciales); filtro «Solo diarios»; un rango sin puntos (por ejemplo, el mes siguiente); páginas 101 y 102. Esperado: página 1 de 101 con 20 puntos; la última página (101) tiene 16; con «Solo diarios», 7 puntos en «Página 1 de 1»; la página 102 viene vacía. |
| CA-9 | Motivos de 9 caracteres («Sin datos») y de 301; motivo válido de 10 («Sin datos.») y de 300; 20 y 21 elementos; elementos de 100 y de 101 caracteres. |

## Decisiones de implementación

- **Vigencia:** depende de la retención de la política del servidor (por defecto 30 días), no es fija (RN-12).
- **Solicitudes pendientes (CA-10):** no puede haber dos PENDIENTE para el mismo servidor; `hayPendiente` lo comprueba y se rechaza con «Ya hay una solicitud pendiente para este servidor» (RN-22).
- **Cómo llegan los errores a la pantalla:** `solicitar` lanza `ValidacionException` con la lista de mensajes de `validar` (se muestran todos, cada uno junto a su campo) o `PermisoDenegadoException`.
- **Página inexistente:** devuelve una lista vacía, no un error (RN-22).
- **Aviso a Dirección:** esta historia no envía correo; Dirección ve las solicitudes pendientes en su lista (HU-015).
- **Largo del motivo:** se cuenta después de quitar los espacios de los extremos (RN-21).
- **«Desde» posterior a «Hasta»:** se rechaza con «La fecha Desde no puede ser posterior a Hasta» (RN-22).
- **«Solo diarios»:** el completo inicial y los diferenciales (RN-22).
- **Punto que vence entre que se abre la pantalla y se envía:** al enviar se vuelve a comprobar la vigencia y se rechaza con «El punto elegido venció. Elegí otro» (RN-22).

## Dependencias, estimación y Definition of Ready

- **Depende de:** HU-008 (los puntos de restauración se generan al verificar los respaldos).
- **Estimación:** 5 SP, Sprint 4 (del 04/01/2027 al 15/01/2027). Va después de HU-008 (Sprint 3), de la que depende, y en el tablero tiene la iteración Sprint 4.

**Chequeo de Definition of Ready:**

- [x] Tiene rol, acción y beneficio, idénticos a los del backlog ([documento 01](../01-vision-y-backlog.md)).
- [x] Tiene 14 criterios Dado/Cuando/Entonces y 10 son casos de error.
- [x] Tiene estimación (5 SP) y prioridad (Must).
- [x] Sus dependencias están planificadas en sprints anteriores o en el mismo sprint (documento 02).
- [x] Indica qué queda fuera y los datos de prueba de cada criterio.
- [x] Revisada por los cuatro integrantes del grupo.
