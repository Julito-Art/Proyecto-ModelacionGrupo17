# ADR-002 – Estilo arquitectónico: monolito modular en capas con puertos y adaptadores

| | |
|---|---|
| **Estado** | Aceptado |
| **Fecha** | 24/10/2026 (revisado el 04/11/2026) |
| **Documentos relacionados** | [04 – RNF y arquitectura](../04-rnf-y-arquitectura.md), [ADR-001](ADR-001-proveedor-de-respaldo-acronis.md) |

## Contexto

Hay que elegir cómo se organiza internamente el SGR. Las condiciones del caso son: una empresa de 30 personas, tres roles de usuario, dos servidores a respaldar, ninguna área de informática y un equipo de desarrollo pequeño.

Los requisitos que condicionan la decisión:

- **RNF-03:** disponibilidad mensual ≥ 99 % (hasta 7,2 horas de caída por mes).
- **RNF-01:** pérdida máxima de datos ≤ 10 minutos en el 99 % de las mediciones; el Planificador no debe perder tareas aunque el sistema se reinicie.
- **RNF-02:** cifrado y comunicaciones seguras; conviene que haya pocos puntos de contacto con el exterior.
- **RNF-06:** el 100 % de las acciones auditadas y consultables en ≤ 3 segundos sobre 12 meses.
- **RNF-07:** interfaz usable sin conocimientos avanzados.
- Restricción **RE-05:** sin área técnica especializada para operarlo.

## Alternativas consideradas

### Alternativa A – Microservicios

Cada capacidad (políticas, respaldo, restauración, alertas, auditoría…) como un servicio independiente.

- **Ventajas:** despliegue y escalado independientes; aislamiento de fallas por servicio.
- **Desventajas:** requiere orquestación, comunicación entre servicios y monitoreo de cada pieza; más puntos de falla, lo que perjudica RNF-03; el costo operativo contradice RE-05; el escalado independiente no aporta nada con 30 usuarios.

### Alternativa B – Arquitectura orientada a eventos con broker de mensajes

Los servicios se comunican publicando y consumiendo eventos a través de un intermediario.

- **Ventajas:** bajo acoplamiento; buena tolerancia a picos y a fallas parciales; natural para alertas y reintentos.
- **Desventajas:** agrega un componente más (el broker) que hay que operar y que se vuelve crítico para RNF-01 y RNF-03; la trazabilidad de lo ocurrido es más difícil (afecta RNF-06); complejidad desproporcionada para el volumen real.

### Alternativa C – Monolito en capas clásico, sin puertos

Una sola aplicación en capas, donde los servicios llaman directamente a la API de Acronis y al correo.

- **Ventajas:** es lo más simple de construir y de operar.
- **Desventajas:** el conocimiento de la API de Acronis se esparce por varios servicios; cambiar de proveedor o ajustarse a un cambio de la API implica tocar muchas partes, y la seguridad de las comunicaciones (RNF-02) se vuelve difícil de controlar en un solo lugar.

## Decisión

Se adopta un **monolito modular en capas (Presentación, Aplicación, Infraestructura) con puertos y adaptadores** para Acronis Cloud (`IProveedorNube` / `AcronisCloudAdapter`) y para el correo (`INotificador` / `NotificadorPorCorreo`). Se despliega como **un único artefacto (`sgr`)**, que contiene la interfaz web y todos los módulos de aplicación en un mismo proceso, con una base de datos SQL propia (`sgr-db`). La interfaz web no es un artefacto separado: se instala, se actualiza y se reinicia junto con la aplicación (ver el diagrama de despliegue del documento 04). Los usuarios acceden mediante una interfaz web.

Es la opción más simple que cumple RNF-03 y RE-05 sin renunciar a aislar lo que más probablemente cambie: el proveedor externo (RNF-02).

## Consecuencias

**Positivas**
- Una sola cosa para desplegar, monitorear y reiniciar (RNF-03, RE-05).
- El Planificador y el estado de las tareas viven juntos, sin coordinación entre procesos (RNF-01).
- Los accesos al exterior están concentrados en dos adaptadores, lo que facilita aplicar y verificar la seguridad (RNF-02).
- Auditoría única y transversal (RNF-06).
- Un equipo pequeño puede construirla, probarla y defenderla.

**Negativas**
- **Despliegue todo-o-nada:** actualizar cualquier módulo reinicia todo el SGR y corta momentáneamente al Planificador (impacto sobre RNF-03).
- **Escalado solo vertical:** si el volumen crece mucho, hay que potenciar el equipo en lugar de agregar instancias.
- **Riesgo de acoplamiento entre módulos** si no se respetan los límites de capas; requiere disciplina y revisión de código.
- **Un único punto de falla:** si cae el servicio, caen a la vez el panel y el Planificador (ver riesgos del documento 04).
