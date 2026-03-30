# 02 — Cola de Mensajes (RabbitMQ)

## Responsabilidades

- Recibe los eventos publicados por la aplicación principal y los distribuye a las tareas automáticas correspondientes que se ejecutan en segundo plano, sin bloquear la respuesta al usuario
- Retiene los mensajes pendientes cuando una tarea automática no está disponible temporalmente y los entrega una vez que el servicio se recupera, garantizando que ningún evento se pierda
- Gestiona los reintentos automáticos de las tareas que no pudieron completarse, sin requerir intervención manual del equipo
- Enruta cada evento al destinatario correcto según su tipo, asegurando que cada tarea reciba únicamente los mensajes que le corresponden

## Por Qué Este Proyecto lo Necesita

Cuando un paciente confirma el pago de su cita, el sistema necesita hacer varias cosas al mismo tiempo: confirmar la cita, bloquear el espacio del médico, enviar el correo de confirmación y programar el recordatorio 24 horas antes. Si todas estas operaciones ocurrieran de forma síncrona, el paciente tendría que esperar hasta que todas terminen antes de recibir una respuesta, lo cual podría tardar varios segundos.

Con RabbitMQ, la aplicación principal solo confirma el pago y publica los eventos correspondientes. Las tareas automáticas los procesan en segundo plano sin que el paciente tenga que esperar. Además, si el servicio de correo falla en ese momento, el mensaje queda guardado en la cola y se reintenta automáticamente cuando el servicio se recupera.

## Elección de Tecnología

| Dimensión | RabbitMQ | Apache Kafka | AWS SQS |
|---|---|---|---|
| **Administrado o instalado** | Instalado por el equipo en un contenedor | Instalado por el equipo — requiere infraestructura adicional | Administrado por AWS |
| **Complejidad operativa** | Baja — configuración simple y bien documentada | Alta — complejo de configurar y operar | Muy baja — AWS lo gestiona todo |
| **Costo para clínica pequeña-mediana** | Gratuito — sin costo de software, solo el servidor donde corre | Alto — requiere más recursos de servidor que RabbitMQ | Variable — se paga por cada mensaje enviado, puede crecer con el volumen |
| **Ventaja principal** | Diseñado específicamente para enrutamiento de mensajes entre servicios | Ideal para procesar millones de eventos por segundo | Sin infraestructura que mantener |

Se elige RabbitMQ porque el volumen de mensajes de una clínica pequeña y mediana — confirmaciones de citas, notificaciones, reportes — es completamente manejable con RabbitMQ. Kafka está diseñado para volúmenes que no aplican a este sistema, y AWS SQS introduce una dependencia de proveedor que no es necesaria cuando RabbitMQ cubre todas las necesidades.

## Trade-offs

| Ventajas | Desventajas |
|---|---|
| Gratuito y con amplia documentación disponible | Requiere ser instalado y mantenido por el equipo |
| Soporta reintentos automáticos y colas de mensajes fallidos | No almacena historial de mensajes como Kafka |
| Configuración simple para el volumen del sistema | Si el servidor donde corre falla, los mensajes podrían perderse sin respaldo |
| Ampliamente usado en sistemas de salud y finanzas | |

## Integración

Como se muestra en el diagrama de despliegue (`diagrams/04-deployment.md`), RabbitMQ se encuentra en la red interna junto a la aplicación principal. La aplicación publica eventos en RabbitMQ cuando ocurre algo importante — un pago confirmado, una cita cancelada, un espacio expirado. Los tres workers del sistema consumen esos eventos de forma independiente: el worker de notificaciones envía correos y SMS, el worker de reportes actualiza los contadores del día, y el worker de espacios expirados libera los espacios que no fueron confirmados a tiempo.
