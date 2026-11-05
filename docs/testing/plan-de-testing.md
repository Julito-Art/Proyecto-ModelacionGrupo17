# Plan de testing del SGR

| | |
|---|---|
| **Entregable** | Punto 9 de la consigna (Parcial 3) |
| **Estado** | Versión final, corregida según la devolución – 04/11/2026 |
| **Casos de prueba** | [HU-003](casos-HU-003.md), [HU-004](casos-HU-004.md), [HU-011](casos-HU-011.md) y [HU-014](casos-HU-014.md) |
| **Base** | [00 – Documento maestro](../00-documento-maestro.md), [04 – RNF](../04-rnf-y-arquitectura.md) y las historias de [`docs/historias/`](../historias/) |

---

## 1. Alcance

**Se prueba:**
- Las cuatro historias del punto 8 (HU-003, HU-004, HU-011 y HU-014), con casos para cada criterio de aceptación.
- Las reglas de negocio que esas historias aplican (RN-03, 04, 05, 06, 07, 08, 10, 12, 13, 16, 17, 19, 20, 21, 22 y 23) y los estados de `TareaDeRespaldo`, `Alerta` y `SolicitudDeRestauracion`.
- Los requisitos no funcionales RNF-01, RNF-06 y RNF-07 en los casos indicados en la sección 3.

**No se prueba en este plan:**
- Las demás historias del backlog (se sigue el mismo método, sin casos todavía).
- La integración con una cuenta real de Acronis Cloud: se usa un doble de prueba de `IProveedorNube` (la API real es un supuesto de modelado).
- Los RNF de rendimiento bajo carga real (RNF-04) y de disponibilidad mensual (RNF-03), que requieren el sistema en funcionamiento.
- Compatibilidad entre navegadores y pruebas de accesibilidad formales.

> **Importante:** el SGR no está implementado, así que los casos están **diseñados, no ejecutados**. Este plan describe cómo se ejecutarían.

## 2. Niveles de la pirámide de pruebas

| Nivel | Qué cubre en el SGR | Proporción objetivo |
|---|---|---|
| **Unitario** | Reglas de una sola clase: `PoliticaDeRespaldo.validar` y `tipoParaCiclo`, `PuntoDeRestauracion.estaVigente`, validaciones de `SolicitudDeRestauracion`, `GestorDeAlertas`. Sin base de datos ni red. | Alrededor del 30 al 40 % |
| **Integración** | Clases que colaboran: planificador, agente, estrategias, adaptador (con un doble de Acronis) y servicio de auditoría; los estados de `TareaDeRespaldo` de punta a punta; el correo con un servidor SMTP de prueba. | Alrededor del 50 % (el nivel con más casos) |
| **E2E** | Flujos completos desde la interfaz web: definir una política, ver una alerta, solicitar una restauración. | Alrededor del 10 al 15 % |
| **Aceptación del usuario** | Prueba de usabilidad con personas sin conocimientos técnicos (RNF-07). | Del 1 al 5 %: una por criterio de usabilidad |

**Distribución real de los 73 casos diseñados:**

| Nivel | HU-003 | HU-004 | HU-011 | HU-014 | Total | Porcentaje |
|---|---|---|---|---|---|---|
| Unitario | 7 | 4 | 1 | 12 | 24 | 33 % |
| Integración | 5 | 10 | 9 | 8 | 32 | 44 % |
| E2E | 6 | 2 | 2 | 6 | 16 | 22 % |
| Aceptación | 1 | 0 | 0 | 0 | 1 | 1 % |

**Esta distribución se aparta de la pirámide clásica** (con más pruebas unitarias que de integración) a propósito: la lógica central del SGR (planificador, agente, estrategias, adaptador y estados de la tarea) es colaboración entre clases, así que la mayor parte de los casos son de integración. Las reglas de una sola clase se cubren con casos unitarios. Los E2E quedan por encima del objetivo (22 % contra 10 a 15 %) porque varios criterios piden algo que solo se ve en pantalla: el mensaje «junto al campo», que la opción no aparezca en el menú, la confirmación o el cambio de la paginación. Esos casos son cortos y se agregaron para que cada criterio tenga un caso que verifique todo su «Entonces».

## 3. Pruebas ligadas a los requisitos no funcionales

| RNF | Caso | Qué verifica | Criterio de aprobación |
|---|---|---|---|
| **RNF-01** (pérdida máxima de datos) | CP-004-11 | Simula **30 días** (la ventana de medición de RNF-01: 43.200 mediciones) con intervalo de 5 minutos, un corte de conexión y una falla con reintentos, y mide cada minuto la antigüedad del último respaldo exitoso | Al menos el 99 % de las mediciones ≤ 10 minutos |
| **RNF-06** (auditabilidad: cobertura) | CP-003-10, CP-004-10, CP-011-09 y CP-014-04 | Cada acción relevante de las historias genera su registro de auditoría | El 100 % de esas acciones deja registro con usuario, fecha y hora exactas (reloj simulado) y resultado |
| **RNF-06** (auditabilidad: tiempo de consulta) | CP-003-19 | Consulta con filtros sobre 12 meses de registros cargados | Respuesta en 3 segundos o menos |
| **RNF-07** (usabilidad) | CP-003-11 | Al menos 5 usuarios de prueba definen una política sin ayuda, cronometrando | Al menos el 90 % en 3 minutos o menos (con 5 usuarios, los 5) |

Pruebas previstas para las demás historias: RNF-02 (rechazo de conexiones con TLS inferior a 1.2), RNF-05 (cronometrar un simulacro de restauración) y RNF-08 (comparar las bases restauradas con las originales).

## 4. Entornos y datos de prueba

| Entorno | Uso | Datos |
|---|---|---|
| **Desarrollo** | Pruebas unitarias y de integración en cada equipo | Datos creados por cada prueba; reloj simulado |
| **Integración** | Pipeline de integración continua | Base de datos de prueba vacía en cada corrida |
| **Aceptación** | E2E y prueba de usabilidad | Dos servidores simulados (STRUMIS y TANGO), un doble de Acronis y un servidor SMTP de prueba |

**Datos de prueba comunes:** usuarios resp.prueba@prometva.test (Responsable de Respaldos), direccion.prueba@prometva.test (Dirección) y gerente.prueba@prometva.test (Gerente de Administración); fecha simulada 26/10/2026; los valores concretos de cada criterio están en la sección *Datos de prueba* de cada historia.

## 5. Herramientas

Como todavía no se eligió el lenguaje de implementación, se nombra el tipo de herramienta:
- Marco de pruebas unitarias del lenguaje elegido (estilo xUnit).
- **Dobles de prueba** de `IProveedorNube` (Acronis) y de `INotificador` (correo), que permiten simular éxito, error, falta de conexión y demoras.
- **Reloj simulado**, para probar intervalos, reintentos y vigencias sin esperar: las pruebas invocan `RelojDelSistema.tic(ahora)` una vez por minuto simulado, igual que en producción, y el Singleton recibe los dobles con `configurar(...)`.
- Servidor SMTP de prueba para verificar los correos.
- Herramienta de pruebas de navegador para los casos E2E.
- Planilla de cronometraje para la prueba de usabilidad.

## 6. Cuándo corren las pruebas

| Momento | Qué corre |
|---|---|
| **En cada commit** | Pruebas unitarias |
| **En el pipeline** (cada solicitud de cambio hacia `main`) | Unitarias e integración |
| **Al cerrar cada sprint** | E2E de las historias del sprint |
| **Antes de cada release** | Todo lo anterior, la prueba de RNF-01 y la prueba de aceptación con usuarios |

## 7. Criterios de entrada y de salida

**Entrada (cuándo se empieza a probar una historia):**
- La historia cumple la Definition of Ready y sus criterios están aprobados.
- Los dobles de prueba y los datos de la historia están preparados.
- El código de la historia está integrado y compila.

**Salida (cuándo se da por aprobada):**
- Todos los casos de la historia pasan, incluidos los negativos y los de valor límite.
- No quedan defectos abiertos de severidad alta.
- Las pruebas ligadas a RNF cumplen su criterio de aprobación.
- Cada criterio de aceptación tiene al menos un caso aprobado (matriz de trazabilidad de cada archivo de casos).

## 8. Trazabilidad general

| Historia | Criterios | Casos | Archivo |
|---|---|---|---|
| HU-003 | 8 | 19 | [casos-HU-003.md](casos-HU-003.md) |
| HU-004 | 9 | 16 | [casos-HU-004.md](casos-HU-004.md) |
| HU-011 | 7 | 12 | [casos-HU-011.md](casos-HU-011.md) |
| HU-014 | 14 | 26 | [casos-HU-014.md](casos-HU-014.md) |
| **Total** | **38** | **73** | |

Los 38 criterios tienen al menos un caso que verifica **todo** su «Entonces».
