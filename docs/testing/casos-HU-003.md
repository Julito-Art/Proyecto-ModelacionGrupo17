# Casos de prueba de HU-003 – Definir la política de respaldo de cada servidor

| | |
|---|---|
| **Historia** | [HU-003](../historias/HU-003-definir-politica-de-respaldo.md) |
| **Plan de testing** | [plan-de-testing.md](plan-de-testing.md) |
| **Estado** | Diseñados (el SGR todavía no está implementado: no hay resultados de ejecución) |

## Casos

| ID | Criterio | Nivel | Tipo | Precondiciones | Pasos | Datos de entrada | Resultado esperado | RNF |
|---|---|---|---|---|---|---|---|---|
| CP-003-01 | CA-1 | Unitario | normal | Ninguna. | Crear `PoliticaDeRespaldo(STRUMIS)` y consultar sus valores. | Servidor STRUMIS. | `intervaloMinutos()` = 5, `retencionDias()` = 30 y `validar()` = verdadero. | — |
| CP-003-02 | CA-1 | E2E | normal | STRUMIS registrado sin política; sesión iniciada como Responsable. | Abrir «Política de respaldo» de STRUMIS. | Usuario resp.prueba@prometva.test. | Intervalo 5, retención 30 y la etiqueta «Sin protección» visibles. | — |
| CP-003-03 | CA-2 | Integración | normal | Sesión de Responsable. | Llamar a `PlanificadorDeRespaldos.definirPolitica` con intervalo 5, retención 30 y hora 23:00; reabrir la política. | Intervalo 5; retención 30; hora 23:00. | La política queda guardada, STRUMIS deja de figurar «Sin protección» y al reabrir se ven 5, 30 y 23:00. | — |
| CP-003-04 | CA-3 | Unitario | negativo | Ninguna. | Crear una política con intervalo 11 y llamar a `validar()`. | Intervalo 11. | `validar()` = falso. | — |
| CP-003-05 | CA-3 | Unitario | límite | Ninguna. | Llamar a `validar()` con intervalos 4, 5, 10 y 11. | Intervalos 4, 5, 10, 11. | Falso, verdadero, verdadero, falso. | — |
| CP-003-06 | CA-3 | E2E | negativo | Sesión de Responsable; política previa de STRUMIS con intervalo 5. | Escribir 11 en el intervalo y guardar. | Intervalo 11. | Mensaje «El intervalo debe estar entre 5 y 10 minutos» junto al campo; la política previa (5) no cambia. | — |
| CP-003-07 | CA-4 | Unitario | límite | Ninguna. | Crear políticas con intervalos 5, 6 y 10; llamar a `validar()` y a `cumpleObjetivoDePerdida()`. | Intervalos 5, 6 y 10. | `validar()` = verdadero en los tres; `cumpleObjetivoDePerdida()` = verdadero con 5 y falso con 6 y con 10. | — |
| CP-003-08 | CA-4 | E2E | límite | Sesión de Responsable. | Guardar una política con intervalo 5; después, una con intervalo 6. | Intervalos 5 y 6. | Con 5: «Política guardada» y ningún aviso. Con 6: la política se guarda y aparece el aviso «Con más de 5 minutos no se garantiza perder como máximo 10 minutos de datos». | — |
| CP-003-09 | CA-5 | Integración | negativo | Sesiones de Gerente de Administración y de Dirección. | Con cada usuario, llamar a `definirPolitica`. | gerente.prueba@prometva.test y direccion.prueba@prometva.test. | Ambas llamadas lanzan `PermisoDenegadoException` con «No tenés permiso para definir políticas» y la política no cambia. | — |
| CP-003-10 | CA-6 | Integración | normal | Guardado exitoso de CP-003-03 con reloj simulado 26/10/2026 10:15:00. | Consultar el registro de auditoría. | — | Existe un registro con usuario resp.prueba@prometva.test, acción CAMBIO_DE_POLITICA, objetivo STRUMIS, resultado «exitoso» y fecha y hora 26/10/2026 10:15:00. | RNF-06 |
| CP-003-11 | CA-2 | Aceptación | normal | Al menos 5 usuarios de prueba sin conocimientos técnicos avanzados (recomendado: 10). | Pedir a cada uno que defina una política de respaldo sin ayuda, cronometrando. | Servidor STRUMIS; valores por defecto. | Al menos el 90 % de los usuarios la define en 3 minutos o menos (con 5 usuarios, los 5). | RNF-07 |
| CP-003-12 | CA-7 | Unitario | límite | Ninguna. | Llamar a `errores()` con retención 6, 7, 90 y 91 (y 0, -1 y 400). | Retenciones 6, 7, 90, 91, 0, -1 y 400. | 7 y 90 sin error; 6, 91, 0, -1 y 400 con «La retención debe estar entre 7 y 90 días». | — |
| CP-003-13 | CA-7 | Unitario | límite | Ninguna. | Llamar a `PoliticaDeRespaldo.esHoraLaboral` y a `errores()` con 07:59, 08:00, 17:59 y 18:00. | Horas 07:59, 08:00, 17:59 y 18:00. | 07:59 y 18:00 sin error; 08:00 y 17:59 con «La hora del diferencial debe estar fuera del horario laboral (08:00 a 18:00)». | — |
| CP-003-14 | CA-7 | E2E | negativo | Sesión de Responsable; política previa de STRUMIS (5, 30, 23:00). | Dejar el intervalo vacío y guardar; repetir con 5,5 y con hora 10:00. | Intervalo «» y 5,5; hora 10:00. | Cada caso muestra su mensaje junto al campo («Ingresá el intervalo como un número entero de minutos», «La hora del diferencial debe estar fuera del horario laboral (08:00 a 18:00)») y la política previa no cambia. | — |
| CP-003-15 | CA-5 | E2E | negativo | Usuarios de Gerente de Administración y de Dirección. | Iniciar sesión con cada uno y revisar el menú. | gerente.prueba@prometva.test y direccion.prueba@prometva.test. | La opción «Política de respaldo» no aparece en el menú de ninguno de los dos. | — |
| CP-003-16 | CA-8 | Unitario | negativo | Ninguna. | Llamar a `errores()` con retención «» y «30,5», y con hora «», «25:00» y «abc». | Retenciones «» y 30,5; horas «», 25:00 y abc. | Las retenciones dan «Ingresá la retención como un número entero de días»; las horas dan «Ingresá la hora del diferencial en formato HH:MM». | — |
| CP-003-17 | CA-8 | Integración | negativo | Sesión de Responsable; política previa de STRUMIS (5, 30, 23:00). | Llamar a `definirPolitica` con intervalo 11 y retención 400 a la vez. | Intervalo 11; retención 400. | Lanza `ValidacionException` con 2 mensajes («El intervalo debe estar entre 5 y 10 minutos» y «La retención debe estar entre 7 y 90 días»); la política guardada sigue siendo (5, 30, 23:00). | — |
| CP-003-18 | CA-8 | E2E | negativo | Sesión de Responsable; política previa de STRUMIS (5, 30, 23:00). | Escribir retención 400 y hora 25:00 y guardar. | Retención 400; hora 25:00. | Se muestran los dos mensajes a la vez, cada uno junto a su campo, y la política previa no cambia. | — |
| CP-003-19 | CA-6 | Integración | normal | Base de auditoría de prueba cargada con 12 meses de registros (unos 450.000: dos servidores con ciclos de 5 minutos). | Consultar la auditoría filtrando por usuario resp.prueba@prometva.test y un mes, cronometrando. | Filtro usuario + rango de 30 días. | La consulta devuelve los registros correctos en 3 segundos o menos. | RNF-06 |

## Matriz de trazabilidad criterio ↔ caso de prueba

| Criterio de aceptación | Casos de prueba |
|---|---|
| CA-1 | CP-003-01, CP-003-02 |
| CA-2 | CP-003-03, CP-003-11 |
| CA-3 | CP-003-04, CP-003-05, CP-003-06 |
| CA-4 | CP-003-07, CP-003-08 |
| CA-5 | CP-003-09, CP-003-15 |
| CA-6 | CP-003-10, CP-003-19 |
| CA-7 | CP-003-12, CP-003-13, CP-003-14 |
| CA-8 | CP-003-16, CP-003-17, CP-003-18 |

## Cobertura

- Criterios de aceptación: 8; criterios con al menos un caso que verifica todo su «Entonces»: 8.
- Casos negativos (error o dato inválido): CP-003-04, CP-003-06, CP-003-09, CP-003-14, CP-003-15, CP-003-16, CP-003-17, CP-003-18.
- Casos de valor límite: CP-003-05, CP-003-07, CP-003-08, CP-003-12, CP-003-13.
- Casos ligados a un RNF: CP-003-10 y CP-003-19 (RNF-06), CP-003-11 (RNF-07).
