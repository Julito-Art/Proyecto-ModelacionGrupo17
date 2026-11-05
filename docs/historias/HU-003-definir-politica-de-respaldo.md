# HU-003 – Definir la política de respaldo de cada servidor

| | |
|---|---|
| **Prioridad (MoSCoW)** | Must |
| **Estimación** | 5 story points |
| **Sprint** | 1 |
| **Épica / Feature** | E1 Respaldo automatizado en la nube / F1.1 Conexión y configuración de respaldos |
| **Depende de** | HU-001 |
| **Origen en OAS** | Alcance del mandato (tiempo real o tiempos prolongados muy cortos); Solución Elegida (plazo mínimo de guardado entre 5 y 10 minutos). |
| **Documentos relacionados** | [06 – Clases](../06-diseno-de-clases.md), [07 – Estados y secuencia](../07-estados-y-secuencia.md), [Casos de prueba](../testing/casos-HU-003.md), [10 – Prototipo y heurísticas](../10-prototipo-y-heuristicas.md) |

## Historia

Como **Responsable de Respaldos** quiero definir para cada servidor el intervalo de respaldo (entre 5 y 10 minutos, por defecto 5), el esquema de tipos de respaldo y la retención para cumplir el objetivo de perder como máximo 10 minutos de datos.

## Contexto y reglas de negocio

- La política define cada cuánto se respalda un servidor y por cuánto tiempo se conservan sus puntos de restauración. Un servidor sin política no se respalda y queda marcado como «Sin protección».
- El valor por defecto de 5 minutos permite cumplir RNF-01: la antigüedad del último respaldo es el intervalo más la duración del respaldo y de su verificación. Con 10 minutos el objetivo puede no cumplirse (SM-03).
- El esquema de tipos de respaldo es fijo (SM-04): un completo inicial único, incrementales en cada intervalo y un diferencial diario fuera del horario laboral. En la pantalla solo se ajusta la hora del diferencial.

**Reglas que aplican** (documento maestro):

- **RN-03:** El intervalo es configurable entre 5 y 10 minutos; el valor por defecto es 5 y con más de 5 se avisa que RNF-01 puede no cumplirse.
- **RN-04:** Esquema de tipos: completo inicial único, incremental en cada intervalo y diferencial diario fuera del horario laboral.
- **RN-12:** Los puntos de restauración se conservan por la retención de la política (por defecto 30 días).
- **RN-19:** Valores de la política: intervalo entero de 5 a 10 minutos; retención entera de 7 a 90 días; hora del diferencial en formato HH:MM; si algo es inválido no se guarda nada.
- **RN-20:** Horario laboral de lunes a viernes de 08:00 a 18:00; el diferencial se programa fuera de ese horario (por defecto 23:00).
- **RN-13:** El cambio de política es una acción relevante y se registra en la auditoría.
- **RN-16:** Solo el Responsable de Respaldos opera: define las políticas.

## Criterios de aceptación

**CA-1 – Valores por defecto y servidor sin protección** · _camino normal_

- **Dado** un servidor STRUMIS registrado que no tiene política y un usuario con rol Responsable de Respaldos autenticado
- **Cuando** abre «Política de respaldo» de STRUMIS
- **Entonces** el intervalo aparece precargado en 5 minutos y la retención en 30 días, y STRUMIS figura como «Sin protección» hasta que se guarde la política

**CA-2 – Guardar una política válida** · _camino normal_

- **Dado** el formulario de STRUMIS con intervalo 5, retención 30 y hora del diferencial 23:00
- **Cuando** el Responsable guarda
- **Entonces** la política queda vigente, STRUMIS deja de figurar como «Sin protección» y al reabrir el formulario se muestran los mismos valores

**CA-3 – Rechazar un intervalo fuera de rango** · _caso de error_

- **Dado** el formulario de STRUMIS con un intervalo de 11 minutos (o de 4)
- **Cuando** el Responsable intenta guardar
- **Entonces** no se guarda nada y, junto al campo, se muestra «El intervalo debe estar entre 5 y 10 minutos»; la política anterior no cambia

**CA-4 – Aviso por intervalo mayor a 5 minutos** · _camino normal_

- **Dado** el formulario de STRUMIS con un intervalo de 6 minutos (o de 10)
- **Cuando** el Responsable guarda
- **Entonces** la política se guarda y se muestra el aviso «Con más de 5 minutos no se garantiza perder como máximo 10 minutos de datos»; con un intervalo de 5 minutos la política se guarda sin ese aviso

**CA-5 – Solo el Responsable define políticas** · _caso de error_

- **Dado** un usuario con rol Gerente de Administración (o Dirección) autenticado
- **Cuando** intenta abrir o guardar una política
- **Entonces** la opción «Política de respaldo» no aparece en su menú y, si lo intenta por otra vía, recibe «No tenés permiso para definir políticas» y la política no cambia

**CA-6 – La acción queda auditada** · _camino normal_

- **Dado** un guardado exitoso de la política de STRUMIS
- **Cuando** el guardado termina
- **Entonces** se registra en la auditoría la acción CAMBIO_DE_POLITICA con el usuario, la fecha y hora del guardado, el servidor y el resultado

**CA-7 – Rechazar valores inválidos de intervalo, retención u hora del diferencial** · _caso de error_

- **Dado** el formulario de STRUMIS con un intervalo vacío o con decimales (5,5), o una retención de 6 o 91 días (también 0, un negativo o 400), o una hora del diferencial de 10:00
- **Cuando** el Responsable intenta guardar
- **Entonces** no se guarda nada y junto a cada campo se muestra «Ingresá el intervalo como un número entero de minutos», «La retención debe estar entre 7 y 90 días» o «La hora del diferencial debe estar fuera del horario laboral (08:00 a 18:00)», según el caso; la política anterior no cambia

**CA-8 – Rechazar una retención o una hora mal escritas, y varios errores a la vez** · _caso de error_

- **Dado** el formulario de STRUMIS con la retención vacía o con decimales (30,5), o con la hora del diferencial vacía o mal escrita («25:00», «abc»)
- **Cuando** el Responsable intenta guardar
- **Entonces** no se guarda nada y junto al campo se muestra «Ingresá la retención como un número entero de días» o «Ingresá la hora del diferencial en formato HH:MM», según el caso; si hay dos o más campos inválidos a la vez, se muestran todos los mensajes juntos, cada uno junto a su campo, y la política anterior no cambia

## Qué queda fuera de esta historia

- Registrar los servidores (HU-001) y conectar con Acronis Cloud (HU-002).
- Lanzar los respaldos según la política (HU-004).
- Aplicar la misma política a los dos servidores de una sola vez.
- Editar el esquema de tipos de respaldo más allá de la hora del diferencial (SM-04).

## Notas de diseño

**Clases del diseño (documento 06) que toca:**

- `PoliticaDeRespaldo`: constantes de intervalo, de retención (`RETENCION_MIN_DIAS`, `RETENCION_MAX_DIAS`) y de horario laboral; el constructor completo `PoliticaDeRespaldo(servidor, intervaloMinutos, retencionDias, horaDiferencial)` carga los valores del formulario; `validar()`, `errores()` (los mensajes de CA-3 y CA-7), `esHoraLaboral(hora)` y `cumpleObjetivoDePerdida()`.
- `PoliticaDeRespaldo(servidor, intervaloMinutos, retencionDias, horaDiferencial)` recibe los **textos** del formulario, para poder distinguir un valor vacío o decimal de uno fuera de rango (CA-7, CA-8).
- `PlanificadorDeRespaldos.definirPolitica(politica, usuario, ahora)` (Singleton): comprueba el rol con `Usuario.tieneRol` y, si no es Responsable, lanza `PermisoDenegadoException` (CA-5); si `validar()` es falso, lanza `ValidacionException` con **todos** los mensajes de `errores()` (CA-3, CA-7, CA-8); si no, guarda la política con `IRepositorioDePoliticas.guardar` y audita el cambio (CA-6). `estaProtegido(servidor)` da la etiqueta «Sin protección» (CA-1, CA-2): se calcula, no se guarda.
- `ServicioDeAuditoria.registrar(...)` con `AccionAuditada.CAMBIO_DE_POLITICA`.
- `Usuario.tieneRol(rol)` para el control de CA-5 y `ServidorOrigen` (`STRUMIS`, `TANGO`).

Esta historia usa el **Singleton**: `definirPolitica` se invoca siempre sobre la única instancia de `PlanificadorDeRespaldos`, de modo que el planificador y la pantalla comparten las mismas políticas.

**Capturas finales de las pantallas** (ya corregidas después de la evaluación heurística, ver el [documento 10](../10-prototipo-y-heuristicas.md#hu-003)):

- [Pantalla 1 – Política de respaldo (formulario)](../../prototipo/HU-003-p1-politica.png)
- [Pantalla 2 – Error de validación del intervalo](../../prototipo/HU-003-p2-error-intervalo.png)

## Datos de prueba

| Criterio | Datos de prueba |
|---|---|
| CA-1 | Servidor STRUMIS sin política; usuario resp.prueba@prometva.test (Responsable de Respaldos). Valores esperados: intervalo 5, retención 30, etiqueta «Sin protección». |
| CA-2 | Intervalo 5; retención 30; hora del diferencial 23:00 (ejemplo de horario fuera de jornada; la hora real se define con la empresa). |
| CA-3 | Intervalos inválidos 11 y 4; intervalos límite válidos 5 y 10; política previa de STRUMIS con intervalo 5 (debe quedar intacta). |
| CA-4 | Intervalos 5 (sin aviso), 6 y 10 (con aviso); retención 30. |
| CA-7 | Intervalo «» y 5,5; retención 6, 7, 90, 91, 0, -1 y 400; hora 07:59 y 18:00 (válidas) y 08:00, 10:00 y 17:59 (inválidas). Política previa con intervalo 5, retención 30 y hora 23:00 (debe quedar intacta). |
| CA-5 | gerente.prueba@prometva.test (Gerente de Administración) y direccion.prueba@prometva.test (Dirección). |
| CA-6 | Guardado de CA-2 con reloj simulado 26/10/2026 10:15:00. Registro esperado: usuario resp.prueba@prometva.test, acción CAMBIO_DE_POLITICA, objetivo STRUMIS, resultado «exitoso», fecha y hora 26/10/2026 10:15:00. |
| CA-8 | Retención «» y 30,5; hora «», «25:00» y «abc»; y a la vez intervalo 11 con retención 400 (dos mensajes). Política previa con intervalo 5, retención 30 y hora 23:00 (debe quedar intacta). |

## Decisiones de implementación

- **Dónde se guarda la política:** el Planificador mantiene las políticas en memoria y las guarda con `IRepositorioDePoliticas.guardar`; al arrancar las carga con `obtener`. Así no se pierden si el servidor se reinicia (RNF-03, ADR-002).
- **Hora del diferencial y día de la semana:** la validación mira solo la hora (RN-20); el diferencial corre todos los días a esa hora.
- **Si cambia la retención:** solo rige para los puntos nuevos; los ya creados conservan su `vigenteHasta` (RN-12).
- **Si cambia el intervalo:** el próximo ciclo se cuenta desde el inicio del último ciclo, no desde el guardado (RN-19).
- **Mensaje de éxito (CA-2):** «Política guardada».
- **Cómo llegan los errores a la pantalla:** con excepciones, no con códigos de retorno. `ValidacionException` lleva la lista de mensajes de `errores()`; la pantalla muestra cada uno junto a su campo. `PermisoDenegadoException` lleva «No tenés permiso para definir políticas».
- **Dirección y la política:** Dirección no accede a la pantalla de políticas (RN-16); ve el estado de los respaldos en el panel, en solo lectura.
- **Falla al guardar:** si `IRepositorioDePoliticas.guardar` falla, se muestra «No se pudo guardar la política. Probá de nuevo», no se audita un cambio exitoso y la política anterior sigue vigente (RN-19).

## Dependencias, estimación y Definition of Ready

- **Depende de:** HU-001 (el servidor debe estar registrado).
- **Estimación:** 5 SP, Sprint 1.

**Chequeo de Definition of Ready:**

- [x] Tiene rol, acción y beneficio, idénticos a los del backlog ([documento 01](../01-vision-y-backlog.md)).
- [x] Tiene 8 criterios Dado/Cuando/Entonces y 4 son casos de error.
- [x] Tiene estimación (5 SP) y prioridad (Must).
- [x] Sus dependencias están planificadas en sprints anteriores o en el mismo sprint (documento 02).
- [x] Indica qué queda fuera y los datos de prueba de cada criterio.
- [x] Revisada por los cuatro integrantes del grupo.
