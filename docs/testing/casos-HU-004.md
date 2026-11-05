# Casos de prueba de HU-004 – Ejecutar respaldos automáticos según la política

| | |
|---|---|
| **Historia** | [HU-004](../historias/HU-004-ejecutar-respaldos-automaticos.md) |
| **Plan de testing** | [plan-de-testing.md](plan-de-testing.md) |
| **Estado** | Diseñados (el SGR todavía no está implementado: no hay resultados de ejecución) |

Todos los casos de integración usan el reloj simulado: `RelojDelSistema.tic(ahora)` se invoca una vez por cada minuto simulado, igual que en producción.

## Casos

| ID | Criterio | Nivel | Tipo | Precondiciones | Pasos | Datos de entrada | Resultado esperado | RNF |
|---|---|---|---|---|---|---|---|---|
| CP-004-01 | CA-1 | Integración | normal | Reloj simulado 14:00; STRUMIS 5 min y TANGO 10 min, con el último ciclo de ambos a las 14:00; Acronis simulado exitoso e íntegro en 90 s. | Avanzar el reloj minuto a minuto hasta las 14:10, registrando los cambios de estado de cada tarea. | Datos de CA-1. | A las 14:05 se crea una tarea de STRUMIS con disparador PROGRAMADO que recorre PROGRAMADA → EN_EJECUCION → VERIFICANDO → COMPLETADA, con un punto de restauración vigente hasta 25/11/2026. Entre 14:01 y 14:09 no se crea ninguna tarea de TANGO; a las 14:10 se crea la primera. | — |
| CP-004-02 | CA-1 | Integración | límite | STRUMIS con intervalo 5 e inicio del último ciclo a las 14:00. | Hacer `tic` a las 14:04, 14:05 y 14:06. Después, con el último ciclo a las 14:05, cambiar el intervalo a 10 a las 14:07 y hacer `tic` a las 14:10 y 14:15. | Tics 14:04, 14:05 y 14:06; cambio de intervalo a las 14:07. | Solo el tic de las 14:05 crea una tarea (14:04 todavía no corresponde y 14:06 ya tiene el ciclo de las 14:05). Con el intervalo nuevo, a las 14:10 no se crea tarea y a las 14:15 sí: el intervalo se cuenta desde el inicio del último ciclo (RN-19). | — |
| CP-004-03 | CA-1 | Integración | normal | Igual que CP-004-01; Acronis simulado falla solo para TANGO. | Avanzar el reloj hasta las 14:10. | Falla simulada en TANGO. | La tarea de STRUMIS no se ve afectada: sigue sus ciclos y llega a COMPLETADA. | — |
| CP-004-04 | CA-2 | Unitario | normal | Agentes A (sin respaldos previos) y B (con completo previo a las 02:00). | Llamar a `PoliticaDeRespaldo.tipoParaCiclo(agente, ahora)` con las 14:05. | Agentes A y B; hora 14:05. | A: COMPLETO. B: INCREMENTAL. | — |
| CP-004-05 | CA-2 | Unitario | límite | Agente B sin diferencial del día; hora del diferencial 23:00. | Llamar a `tipoParaCiclo` a las 22:59, a las 23:00, a las 23:05 (sin diferencial, porque a las 23:00 había un respaldo en curso) y a las 23:30 (con diferencial ya hecho). | Horas 22:59, 23:00, 23:05 y 23:30. | INCREMENTAL, DIFERENCIAL, DIFERENCIAL e INCREMENTAL. | — |
| CP-004-06 | CA-3 | Unitario | negativo | Agente de STRUMIS con `hayRespaldoEnCurso()` = verdadero; corresponde un ciclo. | Llamar a `PlanificadorDeRespaldos.ejecutarCiclo(STRUMIS, ahora)`. | Respaldo en EN_EJECUCION. | Devuelve nulo, no se crea ninguna tarea y se registra CICLO_OMITIDO. | — |
| CP-004-07 | CA-3 | Integración | negativo | Respaldo de STRUMIS en cada estado en curso: EN_EJECUCION, VERIFICANDO, EN_REINTENTO y EN_COLA. | En cada caso, hacer `tic` cada minuto de 14:05 a 14:10. | Los cuatro estados. | En los cuatro casos: 0 tareas nuevas; 1 registro CICLO_OMITIDO a las 14:05, ninguno entre 14:06 y 14:09 y un segundo a las 14:10. | — |
| CP-004-08 | CA-4 | Integración | negativo | Acronis simulado devuelve exitoso = falso con «Acronis: error de autenticación». | Ejecutar el ciclo de STRUMIS. | Motivo «Acronis: error de autenticación». | Tarea en EN_REINTENTO, reintentos = 1, motivo guardado y ningún punto de restauración. | — |
| CP-004-09 | CA-5 | Integración | negativo | Acronis simulado lanza «sin conexión». | Ejecutar el ciclo de TANGO. | Conexión caída. | Tarea en EN_COLA, reintentos = 0 y ninguna alerta. | — |
| CP-004-10 | CA-6 | Integración | normal | Respaldo de CP-004-01. | Consultar la auditoría. | — | Registros RESPALDO_INICIADO (STRUMIS, «iniciado», 26/10/2026 14:05:00) y RESPALDO_FINALIZADO (STRUMIS, «COMPLETADA», 26/10/2026 14:06:30), los dos con el usuario «SGR». | RNF-06 |
| CP-004-11 | CA-1 | Integración | normal | Reloj simulado; STRUMIS con intervalo 5; respaldo más verificación de 90 segundos; un corte de conexión de 20 minutos y una falla con 3 reintentos repartidos en el período. | Simular **30 días** (43.200 tics) y medir cada minuto la antigüedad del último respaldo exitoso. | Intervalo 5 min; duración 90 s; 30 días. | En al menos el 99 % de las 43.200 mediciones la antigüedad es ≤ 10 minutos (como mucho 432 mediciones por encima). | RNF-01 |
| CP-004-12 | CA-7 | Integración | negativo | Tarea de TANGO VERIFICANDO con idRespaldoNube «R-901» a las 14:11. | Hacer que Acronis simulado lance «sin conexión» a las 14:11 y vuelva a responder «íntegro» a las 14:20; hacer `tic` cada minuto. | idRespaldoNube «R-901». | A las 14:11: EN_COLA con «R-901» y reintentos 0. A las 14:20: VERIFICANDO, sin llamadas nuevas a `ejecutarRespaldo` del adaptador, 1 llamada a `validarRespaldo("R-901")`; estado final COMPLETADA con 1 punto de restauración. | — |
| CP-004-13 | CA-8 | Integración | límite | Tarea de STRUMIS VERIFICANDO; validación simulada sin respuesta. | Avanzar el reloj 599 segundos con respuesta «íntegro» en ese instante; repetir sin respuesta hasta los 600 segundos. | Tiempos 599 y 600 s. | A los 599 s: COMPLETADA. A los 600 s: EN_REINTENTO, reintentos 1, motivo «Acronis no respondió: tiempo de espera agotado», sin punto de restauración. | — |
| CP-004-14 | CA-9 | Unitario | negativo | TANGO registrado sin política. | Llamar a `ejecutarCiclo(TANGO, 14:05)`. | Servidor sin política. | Devuelve nulo; 0 tareas, 0 registros de auditoría; `estaProtegido(TANGO)` = falso. | — |
| CP-004-15 | CA-1 | E2E | normal | Entorno de aceptación con reloj simulado a las 14:00; sesión de Responsable; STRUMIS con intervalo 5. | Abrir el panel mínimo, avanzar el reloj hasta las 14:07 y actualizar. | — | A las 14:05 STRUMIS figura «Respaldando (incremental)»; a las 14:07 figura con «último respaldo exitoso: hace 1 min» y próximo ciclo 14:10. | — |
| CP-004-16 | CA-5 | E2E | negativo | Entorno de aceptación; sesión de Responsable. | Cortar la conexión del doble de Acronis antes del ciclo de TANGO y abrir el panel mínimo. | Conexión caída. | TANGO figura «En cola (sin conexión)» y no aparece ninguna alerta. | — |

## Matriz de trazabilidad criterio ↔ caso de prueba

| Criterio de aceptación | Casos de prueba |
|---|---|
| CA-1 | CP-004-01, CP-004-02, CP-004-03, CP-004-11, CP-004-15 |
| CA-2 | CP-004-04, CP-004-05 |
| CA-3 | CP-004-06, CP-004-07 |
| CA-4 | CP-004-08 |
| CA-5 | CP-004-09, CP-004-16 |
| CA-6 | CP-004-10 |
| CA-7 | CP-004-12 |
| CA-8 | CP-004-13 |
| CA-9 | CP-004-14 |

## Cobertura

- Criterios de aceptación: 9; criterios con al menos un caso que verifica todo su «Entonces»: 9.
- Casos negativos (error o dato inválido): CP-004-06, CP-004-07, CP-004-08, CP-004-09, CP-004-12, CP-004-14, CP-004-16.
- Casos de valor límite: CP-004-02, CP-004-05, CP-004-13.
- Casos E2E: CP-004-15, CP-004-16 (las dos pantallas del panel mínimo).
- Casos ligados a un RNF: CP-004-10 (RNF-06), CP-004-11 (RNF-01).
