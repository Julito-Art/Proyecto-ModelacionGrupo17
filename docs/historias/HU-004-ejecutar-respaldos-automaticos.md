# HU-004 – Ejecutar respaldos automáticos según la política

| | |
|---|---|
| **Prioridad (MoSCoW)** | Must |
| **Estimación** | 8 story points |
| **Sprint** | 2 |
| **Épica / Feature** | E1 Respaldo automatizado en la nube / F1.2 Ejecución automática y a demanda |
| **Depende de** | HU-002, HU-003 |
| **Origen en OAS** | Alcance del mandato (de forma automatizada); Diagnóstico (que no se tenga que hacer manualmente); Flujograma de Soluciones (Respaldo Automatizado). |
| **Documentos relacionados** | [06 – Clases](../06-diseno-de-clases.md), [07 – Estados y secuencia](../07-estados-y-secuencia.md), [Casos de prueba](../testing/casos-HU-004.md), [10 – Prototipo y heurísticas](../10-prototipo-y-heuristicas.md) |

## Historia

Como **Responsable de Respaldos** quiero que los respaldos de STRUMIS y TANGO se ejecuten automáticamente según la política de cada servidor para no depender de copias manuales ni de que alguien se acuerde de hacerlas.

## Contexto y reglas de negocio

- Es el corazón del SGR: sin intervención manual, cada servidor se respalda en Acronis Cloud según su política. Cada ejecución es una `TareaDeRespaldo` que recorre los estados definidos en el documento 07.
- La primera ejecución de cada servidor es un respaldo COMPLETO; luego hay incrementales en cada intervalo y un diferencial por día (SM-04). El Planificador elige el tipo en cada ciclo.
- STRUMIS y TANGO se respaldan de forma independiente: cada uno tiene su propio agente (Multiton) y su propia política.

**Reglas que aplican** (documento maestro):

- **RN-01:** El respaldo no detiene ni modifica el uso normal de Strumis ni de Tango.
- **RN-04:** Tipos de respaldo: completo inicial único, incremental en cada intervalo y diferencial diario.
- **RN-05:** Un solo respaldo en ejecución por servidor; si llega un ciclo y hay uno en curso (también EN_COLA), ese ciclo se omite y se registra.
- **RN-06:** Un respaldo que falla por un error propio se reintenta hasta 3 veces (el detalle es de HU-005).
- **RN-13:** Se registran el inicio y el resultado de cada respaldo, y los ciclos omitidos.

## Criterios de aceptación

**CA-1 – Ejecución automática e independiente por servidor** · _camino normal_

- **Dado** STRUMIS con política de 5 minutos, TANGO con política de 10 minutos, conexión con Acronis disponible y el último ciclo de ambos a las 14:00
- **Cuando** son las 14:05 (con una tolerancia de ±1 minuto)
- **Entonces** se crea una tarea de STRUMIS con disparador PROGRAMADO que pasa por EN_EJECUCION y VERIFICANDO hasta COMPLETADA, se genera un punto de restauración vigente por 30 días y no se crea ninguna tarea de TANGO hasta las 14:10

**CA-2 – Tipo de respaldo de cada ciclo** · _camino normal_

- **Dado** un agente sin respaldos previos, y otro con un completo previo
- **Cuando** corre el siguiente ciclo
- **Entonces** el primero hace un respaldo COMPLETO; el segundo hace un INCREMENTAL, salvo que ya sea la hora del diferencial (23:00) y ese día no se haya hecho uno, en cuyo caso hace un DIFERENCIAL

**CA-3 – Ciclo omitido si hay un respaldo en curso** · _caso de error_

- **Dado** un respaldo de STRUMIS que sigue ejecutándose, verificándose, reintentándose o en cola por falta de conexión (EN_COLA)
- **Cuando** llega el siguiente ciclo de STRUMIS
- **Entonces** no se crea otra tarea ni se lanza un segundo respaldo de ese servidor, queda registrado un CICLO_OMITIDO en la auditoría y no se vuelve a intentar hasta el intervalo siguiente (un registro por intervalo, no uno por minuto)

**CA-4 – Falla del respaldo: la tarea pasa a reintento** · _caso de error_

- **Dado** que Acronis informa un error en el primer intento de un respaldo de STRUMIS
- **Cuando** termina ese intento
- **Entonces** la tarea pasa a EN_REINTENTO con reintentos = 1 y el motivo registrado, y no se genera punto de restauración (los reintentos restantes y la alerta corresponden a HU-005 y HU-011)

**CA-5 – Sin conexión a Internet: la tarea queda en cola** · _caso de error_

- **Dado** que no hay conexión a Internet
- **Cuando** se ejecuta el ciclo de TANGO
- **Entonces** la tarea queda EN_COLA sin consumir reintentos y no se genera alerta de falla (el reenvío ordenado al recuperar la conexión corresponde a HU-007)

**CA-6 – Inicio y resultado auditados** · _camino normal_

- **Dado** un respaldo que se inicia y termina
- **Cuando** el planificador procesa su resultado
- **Entonces** se registran RESPALDO_INICIADO al comenzar y RESPALDO_FINALIZADO al terminar, con el servidor, la fecha y hora y el resultado

**CA-7 – Corte de conexión durante la verificación** · _caso de error_

- **Dado** un respaldo de TANGO que ya llegó a la nube y está VERIFICANDO
- **Cuando** se corta Internet y, después, la conexión vuelve
- **Entonces** al cortarse la tarea pasa a EN_COLA conservando el identificador del respaldo en la nube y sin consumir reintentos; al volver la conexión pasa directamente a VERIFICANDO, se repite solo la validación (no se envía otro respaldo) y, si resulta íntegro, termina COMPLETADA con su punto de restauración

**CA-8 – Tiempo de espera agotado en la validación** · _caso de error_

- **Dado** un respaldo de STRUMIS VERIFICANDO cuya validación Acronis no responde
- **Cuando** pasan 10 minutos (600 segundos) sin respuesta
- **Entonces** la tarea pasa a EN_REINTENTO con reintentos = 1 y el motivo «Acronis no respondió: tiempo de espera agotado», y no se genera punto de restauración

**CA-9 – Servidor sin política** · _caso de error_

- **Dado** el servidor TANGO registrado pero sin política (figura «Sin protección»)
- **Cuando** el reloj del sistema ejecuta el ciclo de TANGO
- **Entonces** no se crea ninguna tarea ni se registra nada en la auditoría, y TANGO sigue figurando «Sin protección» en el panel mínimo

## Qué queda fuera de esta historia

- Los reintentos completos, uno por minuto, hasta agotarlos (HU-005).
- Reenviar en orden los respaldos en cola al recuperar la conexión (HU-007).
- La verificación de integridad propiamente dicha (HU-008); acá solo se recorre el estado VERIFICANDO.
- Alertas por falla o atraso (HU-011 y HU-012).
- Lanzar un respaldo a mano (HU-006): usa el mismo flujo pero con disparador MANUAL.

## Notas de diseño

**Clases del diseño (documento 06) que toca:**

- `RelojDelSistema.tic(ahora)`: una vez por minuto llama a `ejecutarCiclo`, `reintentar`, `reanudarEnCola` y `detectarAtrasos`.
- `PlanificadorDeRespaldos` (Singleton): `obtenerInstancia()`, `ejecutarCiclo`, `obtenerPolitica`, `lanzar`, `procesarResultado`, `reanudarEnCola` y `recuperarPendientes`.
- `AgenteDeRespaldo` (Multiton, clave `ServidorOrigen`): `obtenerInstancia(servidor)`, `inicioUltimoCiclo`, `registrarInicioDeCiclo`, `hayRespaldoEnCurso`, `ejecutarRespaldo(tarea, ahora)` y `registrarExito`.
- `PoliticaDeRespaldo` (`proximaEjecucion(inicioUltimoCiclo, ahora)`, `tipoParaCiclo(agente, ahora)`, `retencionDias`).
- `IRepositorioDeTareas` y `IRepositorioDePuntos` (guardan la tarea y el punto de restauración).
- `IEstrategiaDeRespaldo` con `RespaldoCompleto`, `RespaldoIncremental` y `RespaldoDiferencial` (Strategy).
- `IProveedorNube` y `AcronisCloudAdapter` (Adapter) sobre `ApiAcronis`; `SinConexionException`.
- `TareaDeRespaldo` (estados y transiciones del documento 07, incluida `estaEnLaNube` para CA-7), `PuntoDeRestauracion`, `ResultadoDeRespaldo`, `ServicioDeAuditoria` y, desde el Sprint 3, `GestorDeAlertas.cerrarAlertas`.

Esta historia usa **los cuatro patrones**: Singleton (`PlanificadorDeRespaldos`), Multiton (`AgenteDeRespaldo`), Strategy (los tres tipos de respaldo) y Adapter (`AcronisCloudAdapter`). Su secuencia está en el [documento 07](../07-estados-y-secuencia.md).

**Capturas finales de las pantallas** (ya corregidas después de la evaluación heurística, ver el [documento 10](../10-prototipo-y-heuristicas.md#hu-004)):

- [Pantalla 1 – Panel de respaldos (panel mínimo de esta historia)](../../prototipo/HU-004-p1-panel.png)
- [Pantalla 2 – Detalle de una tarea con error y reintento](../../prototipo/HU-004-p2-detalle-tarea.png)

## Datos de prueba

| Criterio | Datos de prueba |
|---|---|
| CA-1 | Reloj simulado 26/10/2026 14:00:00. STRUMIS: intervalo 5 min; TANGO: intervalo 10 min. Acronis simulado responde «exitoso» y «íntegro» en 90 segundos. Tolerancia: se acepta un ciclo entre 14:04:00 y 14:06:00. Punto de restauración esperado: vigente hasta 25/11/2026. |
| CA-2 | Agente A: sin respaldos previos → COMPLETO. Agente B: completo previo a las 02:00; ciclos a las 14:05 → INCREMENTAL; a las 22:59 → INCREMENTAL; a las 23:00 sin diferencial del día → DIFERENCIAL; a las 23:30 con diferencial ya hecho → INCREMENTAL. |
| CA-3 | Respaldo de STRUMIS en cada estado en curso (EN_EJECUCION, VERIFICANDO, EN_REINTENTO y EN_COLA) cuando llega el ciclo de las 14:05. Esperado: 0 tareas nuevas y 1 registro CICLO_OMITIDO; con el respaldo todavía en curso, a las 14:06, 14:07, 14:08 y 14:09 no hay registros nuevos, y a las 14:10 hay un segundo CICLO_OMITIDO. |
| CA-4 | Acronis simulado devuelve exitoso = falso con motivo «Acronis: error de autenticación». Esperado: estado EN_REINTENTO, reintentos 1, motivo guardado, 0 puntos de restauración. |
| CA-5 | Acronis simulado lanza «sin conexión» en el ciclo de TANGO. Esperado: estado EN_COLA, reintentos 0, 0 alertas. |
| CA-6 | Respaldo de CA-1. Registros esperados: RESPALDO_INICIADO (STRUMIS, «iniciado», 26/10/2026 14:05:00) y RESPALDO_FINALIZADO (STRUMIS, «COMPLETADA», 26/10/2026 14:06:30), ambos con el usuario «SGR». |
| CA-7 | Tarea de TANGO VERIFICANDO con idRespaldoNube «R-901» a las 14:11; Acronis simulado lanza «sin conexión» a las 14:11 y vuelve a responder a las 14:20 con «íntegro». Esperado: EN_COLA con «R-901» y reintentos 0; a las 14:20 VERIFICANDO, 0 llamadas nuevas a ejecutarRespaldo del adaptador, 1 llamada a validarRespaldo(«R-901»), estado final COMPLETADA y 1 punto de restauración. |
| CA-8 | Validación simulada que no responde; reloj simulado avanza 600 segundos. Esperado: EN_REINTENTO, reintentos 1, motivo «Acronis no respondió: tiempo de espera agotado», 0 puntos. Borde: con respuesta «íntegro» a los 599 segundos, COMPLETADA. |
| CA-9 | TANGO registrado sin política; reloj en 14:05. Esperado: 0 tareas, 0 registros de auditoría, etiqueta «Sin protección». |

## Decisiones de implementación

- **Quién dispara `ejecutarCiclo`:** `RelojDelSistema` llama a `PlanificadorDeRespaldos.ejecutarCiclo(servidor, ahora)` una vez por minuto para cada servidor. El Planificador obtiene la política (sin política, no hace nada: CA-9) y calcula `proximaEjecucion(inicioUltimoCiclo, ahora)`; si todavía no toca, no hace nada. Si toca, registra el inicio del ciclo en el agente aunque después lo omita, por eso CICLO_OMITIDO aparece una vez por intervalo (CA-3).
- **Diferencial cuando el ciclo no cae a las 23:00:** `tipoParaCiclo` devuelve DIFERENCIAL en el primer ciclo que ocurre a la hora del diferencial **o después**, si ese día todavía no se hizo uno. Si a las 23:00 hay un respaldo en curso, el diferencial se hace en el siguiente ciclo (por ejemplo, 23:05). Si a la medianoche todavía no se pudo hacer, ese día queda sin diferencial y se registra igual en el historial; los incrementales siguen cubriendo RNF-01.
- **`RESPALDO_FINALIZADO`:** se registra solo cuando la tarea llega a un estado final (COMPLETADA o FALLIDA), no después de cada intento ni al quedar EN_COLA.
- **Qué se hace en el Sprint 2 con las alertas:** en el Sprint 2 todavía no existen alertas, así que `procesarResultado` no cierra nada. `GestorDeAlertas.cerrarAlertas` se agrega con HU-011 en el Sprint 3, del mismo modo que la validación real llega con HU-008. La secuencia del documento 07 muestra el comportamiento completo.
- **Panel mínimo:** esta historia incluye la pantalla del panel con el estado de la tarea de cada servidor, el último respaldo exitoso, el próximo ciclo y el detalle de una tarea (pantallas 1 y 2). HU-009 (Sprint 4) completa ese panel con los colores, las alertas, el refresco de 60 segundos y la vista de solo lectura de Dirección.
- **Conteo de reintentos:** `registrarFallo` compara antes de sumar (documento 07): 1 intento inicial + 3 reintentos = 4 intentos; la tarea queda FALLIDA con `reintentos = 3`.
- **Tarea EN_COLA:** el ciclo siguiente se omite y se registra (RN-05).
- **Corte de conexión durante VERIFICANDO (CA-7):** la tarea pasa a EN_COLA conservando `idRespaldoNube` y, al volver la conexión, `reanudarEnCola` la lleva directo a VERIFICANDO (transición EN_COLA → VERIFICANDO del documento 07) y se repite solo la validación. Si el corte fue durante EN_EJECUCION (`idRespaldoNube` nulo), vuelve a EN_EJECUCION y se repite el respaldo.
- **Tiempo de espera de la validación (`TIEMPO_ESPERA_VALIDACION_SEG` = 600):** se trata como respaldo no íntegro: `registrarFallo` con «Acronis no respondió: tiempo de espera agotado»; la tarea pasa a EN_REINTENTO (o FALLIDA si ya no quedan reintentos).
- **CA-1 y la HU-008 (Sprint 3):** en el Sprint 2 la verificación se acepta con un validador simulado del adaptador, que siempre devuelve «íntegro»; la HU-008, en el Sprint 3, lo reemplaza por la validación real de Acronis. Así HU-004 llega a COMPLETADA y se puede aceptar en su sprint.
- **Persistencia y reinicio:** las tareas se guardan con `IRepositorioDeTareas` y los puntos con `IRepositorioDePuntos`. Al reiniciar el servicio, `PlanificadorDeRespaldos.recuperarPendientes(ahora)` lee `pendientes()`, devuelve a cada agente su tarea en curso y su inicio del último ciclo, y una tarea que quedó EN_EJECUCION se trata como un fallo («Reinicio del servicio durante el respaldo»), que consume un reintento.

## Dependencias, estimación y Definition of Ready

- **Depende de:** HU-002 (conexión con Acronis) y HU-003 (política del servidor).
- **Estimación:** 8 SP, Sprint 2.

**Chequeo de Definition of Ready:**

- [x] Tiene rol, acción y beneficio, idénticos a los del backlog ([documento 01](../01-vision-y-backlog.md)).
- [x] Tiene 9 criterios Dado/Cuando/Entonces y 6 son casos de error.
- [x] Tiene estimación (8 SP) y prioridad (Must).
- [x] Sus dependencias están planificadas en sprints anteriores o en el mismo sprint (documento 02).
- [x] Indica qué queda fuera y los datos de prueba de cada criterio.
- [x] Revisada por los cuatro integrantes del grupo.
