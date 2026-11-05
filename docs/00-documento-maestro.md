# 00 – Documento maestro del proyecto

| | |
|---|---|
| **Sistema** | Sistema de Gestión de Respaldos de Prometva (**SGR**) |
| **Caso** | Trabajo de campo de OAS sobre **Prometva** (pyme de ingeniería, fabricación y montaje de estructuras metálicas, 30 personas). El trabajo de OAS fue realizado por otro equipo (Alvarez, Altamirano y Carrizo); este grupo de Modelación (Grupo N° 17: Alvarez Garcia, Flores, Ruiz y Valor) toma ese caso. |
| **Estado** | Versión final, corregida según la devolución – 04/11/2026 |
| **Naturaleza** | Fuente única de nombres, reglas y supuestos. Los documentos 01 a 10 **deben** usar los nombres de este archivo. |

**Convenciones:** los supuestos de modelado llevan el identificador `SM-xx` y la marca **[A VALIDAR]**: son decisiones del grupo que no figuran en el trabajo de campo y deberían confirmarse con la empresa o con el docente.

---

## 1. Descripción del problema

Prometva trabaja a pedido: recibe el pedido del cliente, lo registra en **Strumis** (base de datos general de la empresa, accesible a todo el personal), diseña en Sipe 3D, compra la materia prima y fabrica. La administración y la facturación se registran en **Tango**, con acceso solo de la administradora y de Dirección.

El trabajo de campo detectó que **todos esos datos tienen un único lugar de guardado: un servidor físico privado** (uno para Strumis y otro aparte para Tango). No existe respaldo en la nube ni copia externa. La empresa ya sufrió un ataque que bloqueó su servidor privado y tuvo que pagar un monto elevado para recuperar los datos; hasta hoy no implementó ninguna solución. La falla de una computadora, de un servidor o un nuevo ataque pondrían los datos «en un riesgo muy alto».

### Situación actual (AS-IS) frente a la propuesta (TO-BE)

| Aspecto | Hoy | Con el SGR (propuesta modelada) |
|---|---|---|
| Copias de los datos | Un único lugar: servidor físico privado | Copia automática en Acronis Cloud, además del servidor privado |
| Frecuencia | No hay respaldo externo | Cada 5 a 10 minutos (por defecto, 5) |
| Quién lo hace | No hay respaldo externo, según el relevamiento | Automático; supervisa el Responsable de Respaldos |
| Detección de fallas | No existe | Panel y alertas |
| Recuperación ante un ataque | Pagar para recuperar los datos | Restauración autorizada desde la nube |
| Impacto en la operación | — | Ninguno: los procesos y herramientas actuales no cambian |

> **Importante:** el SGR es una **propuesta modelada**, no algo implementado en la empresa. Lo que hoy existe es solo la columna izquierda.

## 2. Objetivos

**Objetivo general** (del Mandato): disponer de una solución informática que facilite la copia de seguridad de las bases de datos en la nube, para garantizar la seguridad y la máxima protección de los datos y los informes.

**Objetivos específicos:**

1. Respaldar de forma automática las bases de datos de Strumis y Tango en Acronis Cloud con un intervalo de 5 a 10 minutos.
2. Verificar que cada respaldo sea íntegro y utilizable.
3. Detectar y comunicar rápidamente las fallas y atrasos del respaldo.
4. Permitir restaurar los datos de forma controlada y comprobar periódicamente que la restauración funciona.
5. Controlar el acceso y dejar registro de todo lo relevante.
6. No alterar los procesos ni las herramientas actuales de la empresa.

## 3. Alcance y límites

**Dentro del alcance:** programación, ejecución, verificación, monitoreo, alertas, restauración, auditoría e informes de los respaldos de las bases de datos de **Strumis** y **Tango**, usando **Acronis Cloud** como proveedor de almacenamiento.

**Fuera del alcance:**

- Modificar Strumis, Tango o Sipe 3D, o el flujo de procesos de la empresa (**Límite del Mandato**).
- Las soluciones 2 (NAS con sincronización) y 3 (software propio) del trabajo de OAS: se usan solo como alternativas descartadas en los ADR.
- Respaldar archivos que no sean esas bases de datos.
- Desarrollar o programar el sistema: este trabajo solo modela y documenta.

**Límites técnicos del Mandato:** el software usa el mismo lenguaje de base de datos que Strumis (SQL); el respaldo debe ser automático y en tiempos muy cortos.

## 4. Actores y usuarios

### Actores humanos (roles)

| Actor | Descripción | Nota |
|---|---|---|
| **Responsable de Respaldos** | Opera el SGR: configura, supervisa, lanza respaldos y solicita restauraciones. | Rol **a designar por Dirección** (**SM-02**, **[A VALIDAR]**): el organigrama no tiene un área de informática y la empresa indicó que no necesita un área especializada. |
| **Dirección** | Autoriza restauraciones, administra usuarios, consulta auditoría e informes, recibe alertas críticas. | Figura en el organigrama (dos personas de Dirección). |
| **Gerente de Administración** | Consulta, en solo lectura, el estado de los respaldos de TANGO (datos contables y de facturación). | Figura en el organigrama. |

El resto de los 30 empleados **no interactúa con el SGR**: siguen usando Strumis, Tango y Sipe 3D como siempre. Se nombra siempre al rol y no a personas del organigrama.

### Sistemas externos

| Sistema | Rol en el modelo |
|---|---|
| **Acronis Cloud** | Almacena los respaldos en la nube y ejecuta las operaciones de respaldo y restauración. |
| **Servidor Strumis** | Servidor SQL físico privado con la base de Strumis (empleados, órdenes de compra, pedidos, montajes). Origen `STRUMIS`. |
| **Servidor Tango** | Servidor Microsoft SQL físico privado con la base de Tango (facturación, documentos administrativos, datos de nuevos clientes). Origen `TANGO`. |
| **Servicio de correo** | Envía las notificaciones por correo electrónico (**SM-07**). |

> Los servidores de Strumis y Tango **no se comunican con el SGR**: el SGR le ordena a Acronis Cloud y el agente de Acronis instalado en cada servidor envía los datos directamente a la nube. Se los lista como sistemas externos porque son el origen de los datos protegidos.

**Usuario del SGR:** actor general del que heredan los tres roles; se usa en las funciones comunes, como autenticarse.

## 5. Necesidades que se pretende resolver

| N.º | Necesidad | Origen en OAS |
|---|---|---|
| N-1 | Tener una copia de los datos fuera del servidor privado. | Diagnóstico; Mandato |
| N-2 | Que la copia se haga sola, sin intervención manual. | Mandato (alcance); Diagnóstico |
| N-3 | Que se pierda como máximo unos 10 minutos de datos. | Mandato; Solución Elegida (5 a 10 minutos) |
| N-4 | Enterarse rápido si el respaldo falla. | Relevamiento (incidente sufrido) |
| N-5 | Poder recuperar los datos tras una falla o un ataque, sin pagar un rescate. | Relevamiento; Backup Empresarial |
| N-6 | Que solo accedan quienes corresponde. | Estudio Preliminar; Resultado esperado |
| N-7 | No cambiar la forma de trabajar ni requerir personal especializado. | Mandato (límite); Solución Elegida |

## 6. Requisitos funcionales

Cada requisito funcional se corresponde uno a uno con una historia de usuario: **RF-nn = HU-0nn**. Las HU están en [01 – Visión y backlog](01-vision-y-backlog.md).

| RF | Requisito | HU | Épica |
|---|---|---|---|
| RF-01 | Registrar los servidores de origen STRUMIS y TANGO con su agente. | HU-001 | E1 |
| RF-02 | Configurar y probar la conexión con Acronis Cloud. | HU-002 | E1 |
| RF-03 | Definir la política de respaldo de cada servidor. | HU-003 | E1 |
| RF-04 | Ejecutar respaldos automáticos según la política. | HU-004 | E1 |
| RF-05 | Reintentar automáticamente un respaldo fallido. | HU-005 | E1 |
| RF-06 | Lanzar un respaldo manual a demanda. | HU-006 | E1 |
| RF-07 | Encolar respaldos cuando no hay conexión. | HU-007 | E1 |
| RF-08 | Verificar la integridad de cada respaldo. | HU-008 | E1 |
| RF-09 | Consultar el estado de los respaldos en un panel. | HU-009 | E2 |
| RF-10 | Consultar el historial de respaldos con filtros. | HU-010 | E2 |
| RF-11 | Alertar cuando un respaldo falla tras los reintentos. | HU-011 | E2 |
| RF-12 | Alertar por atraso de respaldos. | HU-012 | E2 |
| RF-13 | Notificar a Dirección las alertas críticas. | HU-013 | E2 |
| RF-14 | Solicitar una restauración (total o granular). | HU-014 | E3 |
| RF-15 | Autorizar o rechazar una solicitud de restauración. | HU-015 | E3 |
| RF-16 | Ejecutar una restauración autorizada. | HU-016 | E3 |
| RF-17 | Realizar un simulacro de restauración. | HU-017 | E3 |
| RF-18 | Autenticarse y acceder según el rol. | HU-018 | E4 |
| RF-19 | Gestionar usuarios y roles. | HU-019 | E4 |
| RF-20 | Consultar el registro de auditoría. | HU-020 | E4 |
| RF-21 | Generar el informe mensual de respaldos. | HU-021 | E4 |
| RF-22 | Consultar en solo lectura el estado de los respaldos de TANGO. | HU-022 | E2 |
| RF-23 | Registrar automáticamente las acciones en auditoría. | HU-023 | E4 |

## 7. Requisitos no funcionales (preliminares)

Se desarrollan con escenario completo en [04 – RNF y arquitectura](04-rnf-y-arquitectura.md). Los **identificadores y las métricas de esta tabla son los vinculantes**. Los valores numéricos que no figuran en el trabajo de campo son supuestos **[A VALIDAR]**.

| ID | Atributo | Métrica (preliminar) | Origen |
|---|---|---|---|
| RNF-01 | Confiabilidad (pérdida máxima de datos, RPO) | Con el intervalo por defecto (5 minutos), en el 99 % de las mediciones realizadas durante 30 días, la antigüedad del último respaldo exitoso de cada servidor es ≤ 10 minutos. | Mandato; Solución Elegida (5 a 10 min) |
| RNF-02 | Seguridad (cifrado) | El 100 % de los respaldos se cifra con AES-256 en reposo y viaja por TLS 1.2 o superior; 0 transferencias sin cifrar. | Backup Empresarial (protocolos y cifrado) |
| RNF-03 | Disponibilidad del SGR | Disponibilidad mensual ≥ 99 % (hasta 7,2 horas de caída por mes de 30 días). | Supuesto **SM-08** |
| RNF-04 | Rendimiento (impacto sobre producción) | Con 30 usuarios concurrentes en Strumis, el tiempo de respuesta durante un respaldo no empeora más de un 10 % respecto de la medición sin respaldo. | Mandato (límite: no alterar el flujo) |
| RNF-05 | Recuperabilidad (RTO) | Una restauración total de un servidor (hasta 100 GB) se completa en ≤ 4 horas desde su autorización. | Supuesto **SM-08** |
| RNF-06 | Auditabilidad | El 100 % de las acciones relevantes (lista en RN-13) queda registrado (usuario, fecha y hora, resultado) y se consulta en ≤ 3 segundos sobre 12 meses de registros. | Resultado esperado (confiabilidad) |
| RNF-07 | Usabilidad | El 90 % de al menos 5 usuarios de prueba define una política de respaldo en ≤ 3 minutos y sin ayuda. | Solución Elegida (sin conocimientos avanzados) |
| RNF-08 | Compatibilidad | Después de restaurar en un destino alternativo, el 100 % de las tablas de la base de Strumis y de la de Tango coincide con la original en cantidad de registros y suma de control, y el respaldo no modifica las bases originales. | Mandato (límite: SQL) |

## 8. Reglas de negocio

| ID | Regla |
|---|---|
| RN-01 | **No intrusión.** El respaldo no detiene ni modifica el uso normal de Strumis ni de Tango. |
| RN-02 | **Orígenes.** Hay exactamente dos servidores de origen, con claves `STRUMIS` y `TANGO`, y un agente de respaldo por servidor. |
| RN-03 | **Frecuencia.** El intervalo de respaldo de cada servidor es configurable entre 5 y 10 minutos; el valor por defecto es 5. Con un intervalo mayor a 5 minutos el SGR avisa que el objetivo de RNF-01 puede no cumplirse. |
| RN-04 | **Esquema de tipos.** El primer respaldo de cada servidor es COMPLETO (una sola vez); luego, INCREMENTAL en cada intervalo y DIFERENCIAL una vez por día, fuera del horario laboral (**SM-04**). |
| RN-05 | **Exclusión.** Hay como máximo un respaldo en ejecución por servidor a la vez; si llega un ciclo y el servidor aún tiene un respaldo en curso (ejecutándose, verificándose, reintentándose o en cola por falta de conexión), ese ciclo se omite y se registra. El ciclo omitido cuenta como inicio de ciclo, así que se registra un ciclo omitido por intervalo, no uno por minuto. |
| RN-06 | **Reintentos.** Un respaldo que falla por un error propio del respaldo se reintenta hasta 3 veces, separadas 1 minuto; agotados los reintentos queda FALLIDA y se genera una alerta CRITICA. Las fallas por falta de conexión no consumen reintentos (ver RN-17). |
| RN-07 | **Verificación.** Todo respaldo se verifica antes de pasar a COMPLETADA y de generar un punto de restauración: el SGR pide a Acronis la validación del respaldo y solo acepta el resultado «íntegro» (**SM-13**). |
| RN-08 | **Alertas por falla y atraso.** Más de 15 minutos sin respaldo exitoso genera una alerta de tipo ATRASO y severidad ADVERTENCIA; más de 30 minutos, o un respaldo FALLIDA tras los reintentos (tipo FALLA), genera severidad CRITICA. Hay como máximo una alerta ABIERTA por servidor y tipo: si la situación empeora se eleva su severidad en lugar de crear otra, y pasa a CERRADA cuando un respaldo posterior del servidor resulta exitoso. Si llega una nueva falla con una alerta ABIERTA del mismo servidor y tipo, se actualizan su motivo y su hora, sin reenviar el correo. Una restauración que queda FALLIDA genera una alerta de tipo RESTAURACION_FALLIDA y severidad CRITICA. |
| RN-09 | **Destinatarios.** Las alertas ADVERTENCIA van al Responsable de Respaldos; las CRITICA, al Responsable y a Dirección. |
| RN-10 | **Restauración controlada.** Toda restauración exige solicitud del Responsable y autorización de Dirección; nadie autoriza su propia solicitud. |
| RN-11 | **Red de seguridad.** Antes de restaurar sobre producción, el SGR ejecuta un respaldo previo a restauración del estado actual del servidor (disparador PREVIO_A_RESTAURACION). |
| RN-12 | **Retención.** Los puntos de restauración se conservan por la retención de la política del servidor: entre 7 y 90 días, por defecto 30 (**SM-06**, RN-19). El punto vence con la retención vigente cuando se creó; cambiar la retención no modifica los puntos ya creados, solo los nuevos. |
| RN-13 | **Auditoría.** Toda acción relevante se registra de forma inmutable y se conserva 12 meses (**SM-09**). Acciones relevantes: inicio de sesión (éxito o fallo); alta, baja y cambio de rol de usuarios; registro de servidores; cambio de la conexión con Acronis; cambio de política; respaldos (inicio y resultado, automáticos o manuales); ciclos omitidos por RN-05; alertas generadas; solicitud, autorización, rechazo y ejecución de restauraciones; simulacros; generación de informes. Las acciones que hace el propio sistema (respaldos programados, ciclos omitidos, alertas) se registran con el usuario de sistema «SGR» (`Usuario.SGR`), nunca con un valor vacío. |
| RN-14 | **Simulacros.** Debe hacerse un simulacro de restauración, sobre destino alternativo, al menos cada 90 días (**SM-10**). |
| RN-15 | **Cifrado.** Los datos se cifran con AES-256 en reposo y viajan por TLS 1.2/1.3. |
| RN-16 | **Permisos por rol.** Responsable de Respaldos: opera (registra servidores, define políticas, lanza respaldos) y solicita restauraciones. Dirección: autoriza, administra usuarios, consulta auditoría e informes, y ve el panel de estado de los respaldos en solo lectura (no accede a la pantalla de políticas). Gerente de Administración: solo lectura del estado de TANGO. |
| RN-17 | **Sin conexión.** Si no hay Internet, la tarea queda EN_COLA sin consumir reintentos y sin perderse; se envía en orden cronológico al recuperarse la conexión. |
| RN-18 | **Acceso.** La contraseña tiene al menos 12 caracteres; 5 intentos fallidos bloquean la cuenta 15 minutos. |
| RN-19 | **Valores de la política.** *Intervalo:* número entero de minutos entre 5 y 10 (RN-03); si está vacío o es decimal (por ejemplo 5,5) se rechaza con «Ingresá el intervalo como un número entero de minutos», y si es entero pero fuera de rango, con «El intervalo debe estar entre 5 y 10 minutos». *Retención:* número entero de días entre 7 y 90, por defecto 30; 0, un negativo o 400 se rechazan con «La retención debe estar entre 7 y 90 días», y un valor vacío o decimal, con «Ingresá la retención como un número entero de días». *Hora del diferencial:* formato HH:MM, por defecto 23:00 y fuera del horario laboral (RN-20); vacía o con otro formato (por ejemplo «25:00» o «abc») se rechaza con «Ingresá la hora del diferencial en formato HH:MM». Si hay un valor inválido no se guarda nada y la política anterior no cambia; si hay varios, se muestran todos los mensajes a la vez, cada uno junto a su campo. Si la base de datos no puede guardar la política, se muestra «No se pudo guardar la política. Probá de nuevo» y la política anterior sigue vigente. Al guardar bien se muestra «Política guardada». Cambiar la política no interrumpe un respaldo en curso; el nuevo intervalo se cuenta desde el inicio del último ciclo del servidor (lo guarda su agente). La etiqueta «Sin protección» no se guarda: se calcula, y la tiene todo servidor sin política (**SM-14**). |
| RN-20 | **Horario laboral.** De lunes a viernes, de 08:00 a 18:00 (hora de Argentina, UTC-3) (**SM-14**). La validación mira solo la hora (el diferencial se ejecuta todos los días a la misma hora, incluidos sábados y domingos). La hora del diferencial es válida si es anterior a las 08:00 o igual o posterior a las 18:00 (07:59 y 18:00 son válidas; 08:00 y 17:59, no); si no, se rechaza con «La hora del diferencial debe estar fuera del horario laboral (08:00 a 18:00)». |
| RN-21 | **Datos de una solicitud de restauración.** *Motivo:* se le quitan los espacios de los extremos y el largo se cuenta después de quitarlos; es obligatorio («El motivo es obligatorio» si queda vacío) y tiene entre 10 y 300 caracteres («El motivo debe tener entre 10 y 300 caracteres»). *Elementos* (solo restauración GRANULAR): nombres de tablas o carpetas en texto libre, uno por línea o separados por comas; se quitan los espacios de los extremos de cada elemento y se descartan los vacíos (por ejemplo, entre dos comas seguidas); quedan entre 1 y 20 elementos, de hasta 100 caracteres cada uno y sin repetidos (la comparación no distingue mayúsculas). Mensajes: «Indicá al menos un elemento a restaurar», «Se pueden indicar hasta 20 elementos», «Cada elemento puede tener hasta 100 caracteres» y «Hay elementos repetidos: indicá cada uno una sola vez». En una restauración TOTAL los elementos no se usan: si se escribió alguno, se descarta y no se guarda. *Punto:* es obligatorio («Elegí un punto de restauración»). El SGR no comprueba que existan al solicitar: lo hace Acronis al ejecutar (HU-016) y, si no existen, la restauración queda FALLIDA con ese motivo (**SM-15**). |
| RN-22 | **Elegir un punto de restauración.** Con incrementales cada 5 minutos un servidor puede tener unos 8.600 puntos vigentes, por lo que la pantalla muestra solo los vigentes, del más reciente al más antiguo, en páginas de 20, filtrados por un rango de fechas (por defecto, los últimos 7 días) y con la casilla «Solo diarios» (completo y diferenciales). «Solo diarios» muestra el completo inicial y los diferenciales. Si «Desde» es posterior a «Hasta» se rechaza con «La fecha Desde no puede ser posterior a Hasta». Una página que no existe (por ejemplo, la 102 de 101) devuelve una lista vacía. Si el rango no tiene puntos se muestra «No hay puntos de restauración en el rango elegido»; si el servidor no tiene ninguno vigente, «No hay puntos de restauración vigentes para este servidor». Solo puede haber una solicitud PENDIENTE por servidor («Ya hay una solicitud pendiente para este servidor»). Al enviar se vuelve a comprobar la vigencia: si el punto venció, «El punto elegido venció. Elegí otro». |
| RN-23 | **Tiempos de espera de Acronis.** Cada llamada de control a la API de Acronis (iniciar el respaldo, consultar su estado) espera hasta 30 segundos, y la validación de integridad (RN-07), hasta 10 minutos. El respaldo en sí no tiene tiempo de espera fijo, porque depende del volumen (el completo inicial tarda unas 11 horas). Al agotarse un tiempo de espera, el intento falla con el motivo «Acronis no respondió: tiempo de espera agotado» y consume un reintento (RN-06); no es una falta de conexión (RN-17) (**SM-16**). |

## 9. Supuestos de modelado

| ID | Supuesto | Estado |
|---|---|---|
| SM-01 | Se modela el **SGR** como una capa de gestión que orquesta el respaldo y la restauración y usa **Acronis Cloud como sistema externo**, a través de un adaptador. El trabajo de OAS elige *contratar* Acronis, no desarrollar; para poder modelar clases, patrones y casos de uso se define esta capa. | Decisión asumida por el grupo; no se pudo validar con el docente, se justifica en la defensa |
| SM-02 | Existe el rol **Responsable de Respaldos**, a designar por Dirección. No se asigna a ninguna persona real del organigrama. | **[A VALIDAR]** con la empresa |
| SM-03 | Intervalo por defecto 5 minutos; mínimo 5 y máximo 10 (del plazo «entre 5 y 10 minutos» de la Solución Elegida). El valor por defecto de 5 permite cumplir RNF-01, porque la antigüedad del último respaldo es el intervalo más la duración del respaldo y de su verificación. | **[A VALIDAR]** |
| SM-04 | Esquema: un completo inicial único por servidor, incrementales en cada intervalo y un diferencial diario fuera del horario laboral (Acronis admite los tres métodos según el trabajo de campo). Se descarta el completo semanal: con 100 GB y 20 Mbps de subida tardaría unas 11 horas e impediría los incrementales (RN-05). Si Acronis exigiera completos periódicos, habría que revisar RNF-01. | **[A VALIDAR]** con la documentación de Acronis |
| SM-05 | Reintentos: 3, cada 1 minuto (3 minutos en total). Caben dentro del intervalo mínimo de 5 minutos. | **[A VALIDAR]** |
| SM-06 | Retención de puntos de restauración: 30 días. | **[A VALIDAR]** |
| SM-07 | Las notificaciones se envían por correo electrónico y se muestran en el panel (la empresa ya usa correo en su operación). | **[A VALIDAR]** |
| SM-08 | Volumen supuesto: hasta 100 GB por servidor; enlace a Internet de ≥ 100 Mbps de bajada y ≥ 20 Mbps de subida. El completo inicial (unas 11 horas con 20 Mbps) se hace una sola vez, fuera del horario laboral. El trabajo de campo no da datos de volumen ni de ancho de banda. De ahí salen RNF-03 y RNF-05. | **[A VALIDAR]** |
| SM-09 | Retención de auditoría: 12 meses. | **[A VALIDAR]** |
| SM-10 | Simulacro de restauración cada 90 días. | **[A VALIDAR]** |
| SM-11 | Solo se modela la solución 1 (Acronis Cloud). Las soluciones 2 y 3 del trabajo de OAS son alternativas descartadas en los ADR. | Derivado de OAS |
| SM-12 | El SGR tiene interfaz web y su propia autenticación con usuario y contraseña. | **[A VALIDAR]** |
| SM-13 | La verificación de integridad consiste en pedir a Acronis la validación del respaldo recién creado y aceptar solo el resultado «íntegro». Se asume que Acronis ofrece esa validación; el trabajo de campo no la describe. | **[A VALIDAR]** con la documentación del producto |
| SM-14 | El horario laboral de Prometva es de lunes a viernes, de 08:00 a 18:00; la retención configurable es de 7 a 90 días (el trabajo de campo no da ninguno de los dos datos). «Sin protección» se calcula a partir de la falta de política. | **[A VALIDAR]** con la empresa |
| SM-15 | Los límites de la solicitud de restauración (motivo de 10 a 300 caracteres; hasta 20 elementos de hasta 100 caracteres) y el listado de puntos (páginas de 20, últimos 7 días por defecto) son valores asumidos para que la pantalla sea usable con unos 8.600 puntos. | **[A VALIDAR]** con el Responsable de Respaldos |
| SM-16 | Tiempos de espera de Acronis: 30 s por llamada de control y 10 min para la validación de integridad. La documentación de Acronis puede fijar otros. | **[A VALIDAR]** con la documentación del producto |
| SM-17 | Hay una carga inicial de usuarios al instalar el SGR (tarea técnica T-02): un usuario por rol, con contraseña inicial de al menos 12 caracteres. | **[A VALIDAR]** con la empresa |

## 10. Restricciones

| ID | Restricción | Origen |
|---|---|---|
| RE-01 | No alterar el sistema ni el flujo de procesos actual de la empresa. | Mandato (límite) |
| RE-02 | Usar SQL, el mismo lenguaje de base de datos que Strumis. | Mandato (límite) |
| RE-03 | Respaldo automático, en tiempo real o en tiempos muy cortos. | Mandato (alcance) |
| RE-04 | Calidad y efectividad por encima de lo económico (Diagnóstico), pero con un software de bajo costo y sin área especializada (La Solución Elegida). Se concilian eligiendo el proveedor más económico que cumple los RNF (ADR-001) y limitando el hardware nuevo a un equipo estándar (ADR-003). | Diagnóstico; Solución Elegida |
| RE-05 | Sin requerir un área técnica especializada ni conocimientos avanzados. | Solución Elegida |
| RE-06 | Se requiere conexión a Internet estable y un agente de Acronis en cada servidor. | Backup Empresarial (requisitos) |
| RE-07 | Dos servidores de origen separados, uno por aplicación. | Relevamiento |

## 11. Glosario

| Término | Definición |
|---|---|
| **Prometva** | Pyme de primera generación (15 años) dedicada a ingeniería, fabricación y montaje de estructuras metálicas. |
| **Strumis** | Programa de la empresa (incorporado hace 2 años) con la base de datos general: empleados, órdenes de compra, pedidos y montajes. |
| **Tango** | Programa de administración contable y facturación; acceso restringido. |
| **Servidor físico privado** | Equipo propio de la empresa donde hoy se guardan las bases de datos. |
| **Respaldo** (copia de seguridad) | Copia de los datos hecha para poder recuperarlos. Tipos: **COMPLETO** (todo; solo el primero de cada servidor), **INCREMENTAL** (cambios desde el último respaldo) y **DIFERENCIAL** (cambios desde el último completo). Disparadores: **PROGRAMADO**, **MANUAL** y **PREVIO_A_RESTAURACION**. |
| **Agente de respaldo** | Programa de Acronis instalado en cada servidor, que lee los datos y los envía a la nube. En el modelo de diseño, la clase `AgenteDeRespaldo` es la **representación en el SGR** de ese agente (una por servidor de origen). |
| **Política de respaldo** | Configuración de un servidor: intervalo, esquema de tipos y retención. |
| **Tarea de respaldo** | Una ejecución concreta de un respaldo, con estado y resultado. |
| **Punto de restauración** | Respaldo verificado desde el cual se pueden recuperar datos. |
| **Restauración** | Recuperar datos desde un punto de restauración. **TOTAL**: todo el servidor. **GRANULAR**: solo elementos elegidos (por ejemplo, ciertas tablas). |
| **Simulacro** | Restauración de prueba sobre un destino alternativo, sin tocar producción. |
| **Respaldo previo a restauración** | Respaldo del estado actual de un servidor que se hace justo antes de restaurar sobre producción (RN-11). |
| **RPO** | Pérdida máxima tolerable de datos medida en tiempo. Objetivo: ≤ 10 minutos. |
| **RTO** | Tiempo máximo para recuperar el servicio tras una falla. Objetivo: ≤ 4 horas. |
| **Alerta** | Aviso generado por el SGR ante una falla, un atraso o una restauración fallida. Tipo FALLA, ATRASO o RESTAURACION_FALLIDA; severidad ADVERTENCIA o CRITICA; estado ABIERTA o CERRADA. |
| **Acronis Cloud** | Servicio externo de respaldo en la nube elegido en el trabajo de OAS (en el PDF original aparece también escrito «Acrunis»). |
| **Usuario del SGR** | Actor general del que heredan los tres roles (Responsable de Respaldos, Dirección, Gerente de Administración). |

## 12. Nombres canónicos (vinculantes para todos los documentos)

### Clases e interfaces del modelo de diseño

| Nombre | Rol en el modelo |
|---|---|
| `ServidorOrigen` | Enumeración: `STRUMIS`, `TANGO`. |
| `RelojDelSistema` | Temporizador del SGR: cada minuto invoca al Planificador. Es el actor «Reloj del sistema». |
| `PlanificadorDeRespaldos` | Define las políticas y dispara los respaldos según ellas. **Singleton**, configurable con `configurar(...)`. |
| `AgenteDeRespaldo` | Representación en el SGR del agente de Acronis de un servidor; una instancia por servidor. Candidato a **Multiton** (clave: `ServidorOrigen`). |
| `PoliticaDeRespaldo` | Intervalo, esquema de tipos y retención de un servidor. |
| `IEstrategiaDeRespaldo`, `RespaldoCompleto`, `RespaldoIncremental`, `RespaldoDiferencial` | Algoritmos de respaldo intercambiables. Candidato a **Strategy**. |
| `IProveedorNube`, `AcronisCloudAdapter` | Acceso a Acronis Cloud. Candidato a **Adapter**. |
| `TareaDeRespaldo` | Ejecución de un respaldo. **Entidad central** del diagrama de estados. |
| `PuntoDeRestauracion` | Respaldo verificado disponible para restaurar. |
| `SolicitudDeRestauracion` | Pedido de restauración con su flujo de autorización. |
| `Alerta` | Aviso con tipo, severidad y estado. |
| `INotificador`, `NotificadorPorCorreo` | Envío de notificaciones. |
| `RegistroDeAuditoria` | Registro inmutable de acciones. |
| `Usuario`, `Rol` | Persona que usa el SGR y su rol: `RESPONSABLE_DE_RESPALDOS`, `DIRECCION`, `GERENTE_DE_ADMINISTRACION`. |
| `InformeMensual` | Informe de estado de los respaldos. |
| `GestorDeAlertas` | Genera, actualiza, eleva y cierra alertas y las notifica (componente *Servicio de Alertas*). |
| `ServicioDeAuditoria` | Registra y consulta las acciones (componente *Servicio de Auditoría*). |
| `ServicioDeRestauracion` | Coordina solicitud, autorización, ejecución y simulacro de restauraciones (componente *Servicio de Restauración*). |
| `ResultadoDeSimulacro` | Duración y resultado de un simulacro, comparado con el RTO. |
| `ResultadoDeRespaldo` | Resultado de un respaldo, de su validación o de una restauración en la nube (clase de valor). |
| `ApiAcronis` | API externa de Acronis, adaptada por `AcronisCloudAdapter`; sus operaciones son un supuesto de modelado. |
| `SinConexionException` | Error por falta de conexión a Internet (RN-17). |
| `NotificacionException` | Error al enviar una notificación; la alerta no se pierde (HU-011). |
| `ValidacionException`, `PermisoDenegadoException` | Rechazo de un formulario con la lista de mensajes, y rechazo por rol. |
| `IRepositorioDePoliticas`, `IRepositorioDeTareas`, `IRepositorioDePuntos`, `IRepositorioDeUsuarios`, `IRepositorioDeAlertas`, `IRepositorioDeAuditoria`, `IRepositorioDeSolicitudes`, `IRepositorioDeSimulacros` | Puertos de persistencia, uno por entidad que debe sobrevivir a un reinicio (RNF-03). El de auditoría solo agrega registros (RN-13). |

### Estados

- **`TareaDeRespaldo`:** `PROGRAMADA`, `EN_COLA`, `EN_EJECUCION`, `VERIFICANDO`, `EN_REINTENTO`, `COMPLETADA`, `FALLIDA`.
- **`SolicitudDeRestauracion`:** `PENDIENTE`, `AUTORIZADA`, `RECHAZADA`, `EN_EJECUCION`, `COMPLETADA`, `FALLIDA`.
- **`Alerta`:** `ABIERTA`, `CERRADA`.

### Otros valores

- Nombres de las enumeraciones: `ServidorOrigen`, `TipoDeRespaldo`, `TipoDeDisparo`, `EstadoDeTarea`, `TipoDeRestauracion`, `EstadoDeSolicitud`, `TipoDeAlerta`, `Severidad`, `EstadoAlerta`, `Rol` y `AccionAuditada` (una por cada acción de RN-13, más `CICLO_OMITIDO`).
- Tipo de respaldo: `COMPLETO`, `INCREMENTAL`, `DIFERENCIAL`. Disparador (`TipoDeDisparo`): `PROGRAMADO`, `MANUAL`, `PREVIO_A_RESTAURACION`. Tipo de restauración: `TOTAL`, `GRANULAR`. Tipo de alerta (`TipoDeAlerta`): `FALLA`, `ATRASO`, `RESTAURACION_FALLIDA`. Estado de alerta (`EstadoAlerta`): `ABIERTA`, `CERRADA`. Severidad: `ADVERTENCIA`, `CRITICA`.

## 13. Decisiones de diseño relevantes (preliminares)

Se confirman o ajustan en los documentos 04 a 06.

1. **Patrones candidatos:** Singleton (`PlanificadorDeRespaldos`), Multiton (`AgenteDeRespaldo`), Adapter (`AcronisCloudAdapter`) y Strategy (`IEstrategiaDeRespaldo`). Singleton y Multiton son los patrones del trabajo de la cursada; el Multiton es una variante del Singleton y no figura entre los 23 patrones GoF; como fue asignado por la cátedra se usa igual, y por eso se suman Adapter y Strategy, de otras categorías.
2. **Acronis Cloud** se accede a través de la interfaz `IProveedorNube` para no atar el resto del sistema a un proveedor.
3. **Estilo arquitectónico y despliegue:** se deciden en el documento 04 a partir de los RNF, no antes.

## 14. Criterios de aceptación generales

El modelo se considera aceptable cuando:

1. Cada historia de usuario tiene al menos un caso de uso y cada caso de uso, al menos una historia.
2. Cada RNF tiene métrica medible y está cubierto por al menos una decisión de arquitectura o una prueba.
3. Los nombres de actores, clases, estados y reglas coinciden en todos los documentos con los de este archivo.
4. Todo lo modelado se puede rastrear hasta el trabajo de campo de OAS o a un supuesto **SM-xx** explícito.
5. Ningún documento contradice el Mandato ni sus límites.
6. Un equipo que no conoce el caso puede implementar las 4 HU del punto 8 sin hacer preguntas.

## 15. Fuentes en el trabajo de campo de OAS

Mandato (objetivo, alcance, límite); Estudio Preliminar y modelo de negociación; Entrevista definitoria del mandato; Relevamiento; Organigrama; Diagnóstico; Flujograma de procesos de la empresa; Flujograma de Soluciones; Backup Empresarial (solución 1), Almacenamiento en Red (solución 2) y Software Propio (solución 3); Cuadro comparativo; La Solución Elegida; Resultado esperado.
