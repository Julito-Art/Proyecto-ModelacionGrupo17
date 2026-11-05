# Casos de prueba de HU-014 – Solicitar una restauración

| | |
|---|---|
| **Historia** | [HU-014](../historias/HU-014-solicitar-restauracion.md) |
| **Plan de testing** | [plan-de-testing.md](plan-de-testing.md) |
| **Estado** | Diseñados (el SGR todavía no está implementado: no hay resultados de ejecución) |

## Casos

| ID | Criterio | Nivel | Tipo | Precondiciones | Pasos | Datos de entrada | Resultado esperado | RNF |
|---|---|---|---|---|---|---|---|---|
| CP-014-01 | CA-1 | Unitario | normal | Puntos de TANGO de hace 2, 29 y 31 días. | Filtrar con `PuntoDeRestauracion.estaVigente(ahora)`. | PR-A, PR-B y PR-C; ahora = 26/10/2026 23:00. | Quedan PR-A y PR-B; PR-C se descarta. | — |
| CP-014-02 | CA-1 | Unitario | límite | Un punto con `vigenteHasta = fechaHora + retención de la política` (retención por defecto de 30 días). | Evaluar `estaVigente` con distintas antigüedades. | 29 días 23 h 59 min, exactamente 30 días (`ahora` = `vigenteHasta`) y 30 días 0 h 1 min. | El primero es vigente; el segundo y el tercero están vencidos. | — |
| CP-014-03 | CA-2 | Integración | normal | Sesión de Responsable; punto PR-A. | Llamar a `ServicioDeRestauracion.solicitar` con tipo TOTAL y el motivo indicado. | PR-A; TOTAL; «Pérdida de datos por ataque». | Solicitud PENDIENTE con solicitante, fecha y punto elegido, guardada en `IRepositorioDeSolicitudes`. | — |
| CP-014-04 | CA-2 | Integración | normal | Solicitud creada en CP-014-03 a las 26/10/2026 23:00. | Consultar la auditoría. | — | Registro RESTAURACION_SOLICITADA con usuario resp.prueba@prometva.test y fecha y hora 26/10/2026 23:00. | RNF-06 |
| CP-014-05 | CA-3 | Unitario | negativo | Ninguna. | Llamar a `SolicitudDeRestauracion.validar` con motivo vacío y con motivo de tres espacios. | Motivos «» y «   ». | Ambos devuelven «El motivo es obligatorio»; `solicitar` lanza `ValidacionException` y no se crea solicitud. | — |
| CP-014-06 | CA-4 | Unitario | negativo | Tipo GRANULAR. | Llamar a `validar` sin elementos. | Texto de elementos vacío. | Devuelve «Indicá al menos un elemento a restaurar» y no se crea solicitud. | — |
| CP-014-07 | CA-4 | Unitario | límite | Tipo GRANULAR. | Llamar a `validar` con 0 y con 1 elemento. | 0 elementos y «Clientes». | Con 0 se rechaza; con 1 se acepta. | — |
| CP-014-08 | CA-5 | Integración | normal | Sesión de Responsable. | Solicitar GRANULAR con dos elementos. | «Facturas 2026» y «Clientes»; motivo «Recuperar facturación eliminada por error». | Solicitud PENDIENTE que conserva los dos elementos y el motivo. | — |
| CP-014-09 | CA-6 | Integración | negativo | Sesiones de Dirección y de Gerente de Administración. | Intentar `solicitar` con cada usuario. | direccion.prueba@prometva.test y gerente.prueba@prometva.test. | Ambas lanzan `PermisoDenegadoException` con «No tenés permiso para solicitar restauraciones» y no se crea nada. | — |
| CP-014-10 | CA-7 | Integración | negativo | STRUMIS sin puntos vigentes. | Llamar a `contarPuntosVigentes(STRUMIS, …)`. | Puntos de más de 30 días. | Devuelve 0 y la pantalla queda en el estado vacío «No hay puntos de restauración vigentes para este servidor». | — |
| CP-014-11 | CA-2 | E2E | normal | Sesión de Responsable; TANGO con PR-A y PR-B. | Abrir la solicitud, elegir PR-A, TOTAL, escribir el motivo y enviar. | PR-A; TOTAL; «Pérdida de datos por ataque». | Se recorren las pantallas 1 y 3: la confirmación muestra el número de solicitud, el estado PENDIENTE, el servidor, el punto, el tipo y el motivo. | — |
| CP-014-12 | CA-8 | Unitario | normal | Repositorio de puntos simulado: STRUMIS con 2.016 puntos en los últimos 7 días. | Llamar a `puntosVigentes` y `contarPuntosVigentes` con la página 1. | Rango de 7 días; página 1. | 20 puntos, del más reciente al más antiguo, y 101 páginas en total. | — |
| CP-014-13 | CA-8 | Unitario | límite | Mismos datos que CP-014-12. | Pedir las páginas 101 y 102. | Páginas 101 y 102. | La 101 trae 16 puntos; la 102 viene vacía, sin error. | — |
| CP-014-14 | CA-8 | Integración | normal | STRUMIS con incrementales, 1 diferencial por día y el completo inicial. | Llamar a `contarPuntosVigentes` y `puntosVigentes` con `soloDiarios` verdadero en los últimos 7 días, y con un rango sin puntos. | `soloDiarios` = verdadero; rango del mes siguiente. | Con «Solo diarios»: 7 puntos, todos diferenciales, en 1 página (la cantidad de páginas pasa de 101 a 1). El rango vacío devuelve 0 puntos y la pantalla muestra «No hay puntos de restauración en el rango elegido». | — |
| CP-014-15 | CA-9 | Unitario | límite | Ninguna. | Llamar a `SolicitudDeRestauracion.validar` con motivos de 9, 10, 300 y 301 caracteres. | «Sin datos» (9), «Sin datos.» (10), 300 y 301 caracteres. | 10 y 300 sin error; 9 y 301 con «El motivo debe tener entre 10 y 300 caracteres». | — |
| CP-014-16 | CA-9 | Unitario | límite | Tipo GRANULAR. | Llamar a `validar` con 20 y 21 elementos, y con elementos de 100 y de 101 caracteres. | 20 y 21 elementos; 100 y 101 caracteres. | 20 elementos y 100 caracteres sin error; 21 con «Se pueden indicar hasta 20 elementos» y 101 con «Cada elemento puede tener hasta 100 caracteres». | — |
| CP-014-17 | CA-7 | E2E | negativo | Sesión de Responsable; STRUMIS sin puntos vigentes. | Abrir la solicitud y elegir STRUMIS. | Puntos de más de 30 días. | Se muestra la pantalla 4 con «No hay puntos de restauración vigentes para este servidor» y el botón de envío deshabilitado. | — |
| CP-014-18 | CA-1 | E2E | normal | Sesión de Responsable; TANGO con PR-A, PR-B y PR-C; fecha simulada 26/10/2026 23:00. | Abrir la solicitud y elegir TANGO; después cambiar «Desde» a 21/09/2026. | Rango por defecto y rango ampliado. | Con el rango por defecto se lista solo PR-A; con el rango ampliado, PR-A y PR-B; PR-C no aparece en ningún caso. | — |
| CP-014-19 | CA-3 | E2E | negativo | Sesión de Responsable. | Dejar el motivo vacío y enviar. | Motivo «». | El mensaje «El motivo es obligatorio» aparece junto al campo Motivo y no se crea ninguna solicitud. | — |
| CP-014-20 | CA-6 | E2E | negativo | Usuarios de Dirección y de Gerente de Administración. | Iniciar sesión con cada uno y revisar el menú. | direccion.prueba@prometva.test y gerente.prueba@prometva.test. | La opción «Nueva solicitud de restauración» no aparece en el menú de ninguno. | — |
| CP-014-21 | CA-10 | Integración | negativo | Solicitud S-12 de TANGO en PENDIENTE. | Llamar a `solicitar` para TANGO con PR-A; después, para STRUMIS. | PR-A de TANGO; un punto vigente de STRUMIS. | La de TANGO lanza `ValidacionException` con «Ya hay una solicitud pendiente para este servidor» y no se crea; la de STRUMIS queda PENDIENTE. | — |
| CP-014-22 | CA-11 | Unitario | negativo | Ninguna. | Llamar a `puntosVigentes` con «Desde» posterior a «Hasta». | Desde 26/10/2026; Hasta 19/10/2026. | Lanza `ValidacionException` con «La fecha Desde no puede ser posterior a Hasta» y no devuelve puntos: la pantalla conserva la lista anterior y muestra el mensaje junto al filtro. | — |
| CP-014-23 | CA-12 | Integración | negativo | Punto con `vigenteHasta` 26/10/2026 23:00:00, ofrecido en la lista a las 22:58. | Enviar la solicitud con reloj simulado a las 23:00:05. | Punto que vence durante la carga del formulario. | Lanza `ValidacionException` con «El punto elegido venció. Elegí otro» y no se crea ninguna solicitud. | — |
| CP-014-24 | CA-13 | Unitario | negativo | Ninguna. | Llamar a `separarElementos` y `validar` con los textos indicados. | GRANULAR «Clientes, clientes, Facturas 2026, , »; GRANULAR «Clientes, , Facturas 2026»; TOTAL «Clientes». | El primero da «Hay elementos repetidos: indicá cada uno una sola vez»; el segundo se acepta con 2 elementos (el vacío se descarta); el TOTAL se acepta y la solicitud queda con 0 elementos. | — |
| CP-014-25 | CA-14 | Unitario | negativo | Ninguna. | Llamar a `validar` sin punto. | Punto nulo; TOTAL; motivo «Pérdida de datos por ataque». | Devuelve «Elegí un punto de restauración» y no se crea ninguna solicitud. | — |
| CP-014-26 | CA-8 | E2E | normal | Sesión de Responsable; STRUMIS con los puntos de CP-014-12. | Abrir la solicitud, elegir STRUMIS y marcar «Solo diarios». | — | Antes: «Página 1 de 101». Después: 7 diferenciales y «Página 1 de 1». | — |

## Matriz de trazabilidad criterio ↔ caso de prueba

| Criterio de aceptación | Casos de prueba |
|---|---|
| CA-1 | CP-014-01, CP-014-02, CP-014-18 |
| CA-2 | CP-014-03, CP-014-04, CP-014-11 |
| CA-3 | CP-014-05, CP-014-19 |
| CA-4 | CP-014-06, CP-014-07 |
| CA-5 | CP-014-08 |
| CA-6 | CP-014-09, CP-014-20 |
| CA-7 | CP-014-10, CP-014-17 |
| CA-8 | CP-014-12, CP-014-13, CP-014-14, CP-014-26 |
| CA-9 | CP-014-15, CP-014-16 |
| CA-10 | CP-014-21 |
| CA-11 | CP-014-22 |
| CA-12 | CP-014-23 |
| CA-13 | CP-014-24 |
| CA-14 | CP-014-25 |

## Cobertura

- Criterios de aceptación: 14; criterios con al menos un caso que verifica todo su «Entonces»: 14.
- Casos negativos (error o dato inválido): CP-014-05, CP-014-06, CP-014-09, CP-014-10, CP-014-17, CP-014-19, CP-014-20, CP-014-21, CP-014-22, CP-014-23, CP-014-24, CP-014-25.
- Casos de valor límite: CP-014-02, CP-014-07, CP-014-13, CP-014-15, CP-014-16.
- Casos ligados a un RNF: CP-014-04 (RNF-06).
