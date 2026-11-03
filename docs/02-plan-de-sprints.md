# 02 – Plan de sprints

| | |
|---|---|
| **Entregable** | Punto 2 de la consigna (Parcial 3) |
| **Estado** | Borrador v2 – 24/10/2026 |
| **Backlog de origen** | [01 – Visión y backlog](01-vision-y-backlog.md) |

## 1. Duración y capacidad

- **Duración del sprint:** 2 semanas (10 días hábiles). Es lo bastante corta para recibir feedback y lo bastante larga para cerrar HU de 3 a 8 puntos.
- **Equipo supuesto:** 4 integrantes con dedicación parcial (estudiantes).
- **Capacidad supuesta:** **20 story points por sprint** = 4 personas × 5 SP por persona y sprint.
- **Criterio:** se calcula para una dedicación de unas 10 horas semanales por persona, tomando como referencia que 1 SP equivale a medio día de trabajo de una persona. Como no hay historial de velocidad, el valor es una **suposición a recalibrar** después del Sprint 1. Ningún sprint se carga al 100 % para dejar margen a reuniones y retrabajo (la carga va de 16 a 19 puntos, salvo el sprint de cierre).
- **Receso:** no se agenda ningún sprint entre el 21/12/2026 y el 01/01/2027 (fiestas de fin de año).
- **Regla de planificación:** ningún sprint supera la capacidad; las dependencias se respetan (una HU nunca va antes de lo que necesita).

## 2. Escala de estimación

Secuencia de Fibonacci acotada (1, 2, 3, 5, 8). **HU de referencia:** HU-001 (registrar servidores) = 3 SP: un formulario con validaciones y persistencia. Una HU de 8 SP implica integración con sistemas externos o varios estados.

## 3. Resumen de sprints

| Sprint | Fechas (supuestas) | Objetivo (sprint goal) | HU | Puntos | Capacidad |
|---|---|---|---|---|---|
| 1 | 09/11/2026 – 20/11/2026 | Dejar listos el acceso por rol, el registro de auditoría, los servidores STRUMIS y TANGO y la política de respaldo de cada uno. | HU-018, HU-023, HU-001, HU-003 | 16 | 20 |
| 2 | 23/11/2026 – 04/12/2026 | Lograr que ambos servidores se respalden automáticamente en Acronis Cloud, con reintentos y con la opción de lanzar un respaldo manual. | HU-002, HU-004, HU-005, HU-006 | 19 | 20 |
| 3 | 07/12/2026 – 18/12/2026 | Garantizar que cada respaldo sea íntegro, que no se pierda ante cortes de Internet y que las fallas y los atrasos generen alertas. | HU-008, HU-011, HU-007, HU-012 | 18 | 20 |
| 4 | 04/01/2027 – 15/01/2027 | Dar visibilidad del estado y del historial de los respaldos y habilitar la solicitud y la autorización de restauraciones. | HU-009, HU-014, HU-015, HU-013, HU-010 | 18 | 20 |
| 5 | 18/01/2027 – 29/01/2027 | Poder recuperar datos y comprobarlo con un simulacro, y auditar lo ocurrido. | HU-016, HU-017, HU-020 | 16 | 20 |
| 6 | 01/02/2027 – 12/02/2027 | Cerrar el producto con la gestión de usuarios, el informe mensual y la consulta de solo lectura para el Gerente de Administración. | HU-019, HU-021, HU-022 | 8 | 20 |

Total planificado: **95 SP** en 6 sprints (capacidad total 120 SP). Las fechas pertenecen al proyecto hipotético de implementación y se pueden mover; lo importante es el orden.

## 4. Detalle por sprint

### Sprint 1

**Objetivo:** Dejar listos el acceso por rol, el registro de auditoría, los servidores STRUMIS y TANGO y la política de respaldo de cada uno.

| Orden | HU | Título | MoSCoW | SP | Depende de |
|---|---|---|---|---|---|
| 1 | HU-018 | Autenticarse y acceder según el rol | Must | 5 | — |
| 2 | HU-023 | Registrar automáticamente las acciones en auditoría | Must | 3 | HU-018 |
| 3 | HU-001 | Registrar los servidores de origen | Must | 3 | HU-018 |
| 4 | HU-003 | Definir la política de respaldo de cada servidor | Must | 5 | HU-001 |

Puntos del sprint: **16** de 20 (holgura 4).

### Sprint 2

**Objetivo:** Lograr que ambos servidores se respalden automáticamente en Acronis Cloud, con reintentos y con la opción de lanzar un respaldo manual.

| Orden | HU | Título | MoSCoW | SP | Depende de |
|---|---|---|---|---|---|
| 1 | HU-002 | Conectar el sistema con Acronis Cloud | Must | 5 | HU-018 |
| 2 | HU-004 | Ejecutar respaldos automáticos según la política | Must | 8 | HU-002, HU-003 |
| 3 | HU-005 | Reintentar automáticamente un respaldo fallido | Must | 3 | HU-004 |
| 4 | HU-006 | Lanzar un respaldo manual a demanda | Should | 3 | HU-004, HU-023 |

Puntos del sprint: **19** de 20 (holgura 1).

### Sprint 3

**Objetivo:** Garantizar que cada respaldo sea íntegro, que no se pierda ante cortes de Internet y que las fallas y los atrasos generen alertas.

| Orden | HU | Título | MoSCoW | SP | Depende de |
|---|---|---|---|---|---|
| 1 | HU-008 | Verificar la integridad de cada respaldo | Must | 5 | HU-004 |
| 2 | HU-011 | Recibir una alerta cuando un respaldo falla | Must | 5 | HU-005 |
| 3 | HU-007 | Encolar respaldos cuando no hay conexión a Internet | Should | 5 | HU-004 |
| 4 | HU-012 | Recibir una alerta por atraso de respaldos | Should | 3 | HU-011 |

Puntos del sprint: **18** de 20 (holgura 2).

### Sprint 4

**Objetivo:** Dar visibilidad del estado y del historial de los respaldos y habilitar la solicitud y la autorización de restauraciones.

| Orden | HU | Título | MoSCoW | SP | Depende de |
|---|---|---|---|---|---|
| 1 | HU-009 | Consultar el estado de los respaldos en un panel | Must | 5 | HU-004 |
| 2 | HU-014 | Solicitar una restauración | Must | 5 | HU-008 |
| 3 | HU-015 | Autorizar o rechazar una solicitud de restauración | Must | 3 | HU-014 |
| 4 | HU-013 | Notificar a Dirección las alertas críticas | Should | 2 | HU-011, HU-012 |
| 5 | HU-010 | Consultar el historial de respaldos con filtros | Should | 3 | HU-004 |

Puntos del sprint: **18** de 20 (holgura 2).

### Sprint 5

**Objetivo:** Poder recuperar datos y comprobarlo con un simulacro, y auditar lo ocurrido.

| Orden | HU | Título | MoSCoW | SP | Depende de |
|---|---|---|---|---|---|
| 1 | HU-016 | Ejecutar una restauración autorizada | Must | 8 | HU-015 |
| 2 | HU-017 | Realizar un simulacro de restauración | Should | 5 | HU-016 |
| 3 | HU-020 | Consultar el registro de auditoría | Should | 3 | HU-023 |

Puntos del sprint: **16** de 20 (holgura 4).

### Sprint 6

**Objetivo:** Cerrar el producto con la gestión de usuarios, el informe mensual y la consulta de solo lectura para el Gerente de Administración.

| Orden | HU | Título | MoSCoW | SP | Depende de |
|---|---|---|---|---|---|
| 1 | HU-019 | Gestionar usuarios y roles | Should | 3 | HU-018, HU-023 |
| 2 | HU-021 | Generar el informe mensual de respaldos | Could | 3 | HU-010 |
| 3 | HU-022 | Consultar el estado de los respaldos de Tango | Could | 2 | HU-009, HU-018 |

Puntos del sprint: **8** de 20 (holgura 12).

## 5. Verificación de dependencias

| HU | Sprint | Depende de (sprint) | ¿Se respeta? |
|---|---|---|---|
| HU-023 | 1 | HU-018 (S1) | Sí |
| HU-001 | 1 | HU-018 (S1) | Sí |
| HU-003 | 1 | HU-001 (S1) | Sí |
| HU-002 | 2 | HU-018 (S1) | Sí |
| HU-004 | 2 | HU-002 (S2), HU-003 (S1) | Sí |
| HU-005 | 2 | HU-004 (S2) | Sí |
| HU-006 | 2 | HU-004 (S2), HU-023 (S1) | Sí |
| HU-008 | 3 | HU-004 (S2) | Sí |
| HU-011 | 3 | HU-005 (S2) | Sí |
| HU-007 | 3 | HU-004 (S2) | Sí |
| HU-012 | 3 | HU-011 (S3) | Sí |
| HU-009 | 4 | HU-004 (S2) | Sí |
| HU-014 | 4 | HU-008 (S3) | Sí |
| HU-015 | 4 | HU-014 (S4) | Sí |
| HU-013 | 4 | HU-011 (S3), HU-012 (S3) | Sí |
| HU-010 | 4 | HU-004 (S2) | Sí |
| HU-016 | 5 | HU-015 (S4) | Sí |
| HU-017 | 5 | HU-016 (S5) | Sí |
| HU-020 | 5 | HU-023 (S1) | Sí |
| HU-019 | 6 | HU-018 (S1), HU-023 (S1) | Sí |
| HU-021 | 6 | HU-010 (S4) | Sí |
| HU-022 | 6 | HU-009 (S4), HU-018 (S1) | Sí |

Dentro de un mismo sprint, el orden de la tabla del punto 4 es el orden de trabajo, de modo que cada HU se hace después de la que necesita. Las HU sin dependencias declaradas (HU-018) son las primeras del Sprint 1; todas las demás funciones requieren iniciar sesión, por lo que HU-018 va primero.

## 6. Hitos de producto

- **Cierre del Sprint 3:** el núcleo de protección funciona: respaldo automático (HU-004), reintentos (HU-005), verificación de integridad (HU-008), cola ante cortes de Internet (HU-007) y alertas de falla y atraso (HU-011, HU-012).
- **Cierre del Sprint 4:** hay panel, historial y solicitud y autorización de restauraciones.
- **Cierre del Sprint 5:** quedan entregadas **todas las HU de prioridad Must**, incluida la restauración (HU-016).
- **Sprint 6:** HU Should y Could restantes. Si hay que recortar alcance, se recorta el Sprint 6.

## 7. Carga en el tablero de GitHub (Sprint, Puntos y Prioridad)

Por indicación del docente se usa GitHub en lugar de Azure DevOps. El backlog vive en **issues del repositorio** y en un **tablero (GitHub Project)** con tres campos personalizados, con estos nombres exactos:

| Campo del tablero | Tipo | Valores |
|---|---|---|
| **Sprint** | Iteration | Seis iteraciones de 2 semanas (tabla de abajo) |
| **Puntos** | Number | Story points de cada HU (escala 1, 2, 3, 5, 8) |
| **Prioridad** | Single select | Must, Should, Could, Won't (MoSCoW) |

Las seis iteraciones del campo **Sprint**:

| Iteración | Inicio | Fin |
|---|---|---|
| Sprint 1 | 2026-11-09 | 2026-11-20 |
| Sprint 2 | 2026-11-23 | 2026-12-04 |
| Sprint 3 | 2026-12-07 | 2026-12-18 |
| Sprint 4 | 2027-01-04 | 2027-01-15 |
| Sprint 5 | 2027-01-18 | 2027-01-29 |
| Sprint 6 | 2027-02-01 | 2027-02-12 |

Entre el Sprint 3 y el Sprint 4 hay un receso de dos semanas (21/12/2026 al 03/01/2027): en el campo Sprint se carga como un *break* entre iteraciones.

Las épicas, features e historias se cargan como issues con las etiquetas `épica`, `feature` e `historia`, enlazadas como sub-issues (feature dentro de épica, historia dentro de feature). La creación de etiquetas e issues se puede automatizar con el script que acompaña este trabajo; el detalle está en `_interno/guia-github.md`. Los valores de Sprint, Puntos y Prioridad de cada HU son los de las secciones 3 y 4 de este documento y los del backlog.

## 8. Riesgos y supuestos del plan

- La velocidad de 20 SP es una suposición sin historial; si en el Sprint 1 se completan menos puntos, se mueven HU Should/Could del Sprint 6 fuera del plan.
- HU-004 (8 SP) concentra la mayor complejidad (ejecución automática con integración a Acronis); si se atrasa, arrastra a HU-005, HU-006, HU-008 y a todo el Sprint 3. Está prevista su división en dos si se desborda.
- Las HU de los Sprints 2 a 5 asumen la integración con Acronis Cloud definida en HU-002; un cambio de proveedor impactaría el plan (ver ADR-001).
