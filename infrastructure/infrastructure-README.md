# Infrastructure — Componentes de Infraestructura

Esta carpeta describe los 8 componentes de infraestructura del Healthcare Scheduling System. Cada archivo explica qué hace el componente en el sistema, por qué se necesita, qué tecnología se eligió y cómo se conecta con los demás.

| Documento | Componente | Tecnología |
|---|---|---|
| [01-load-balancer.md](01-load-balancer.md) | Balanceador de carga | AWS ALB |
| [02-message-queue.md](02-message-queue.md) | Cola de mensajes | RabbitMQ |
| [03-background-workers.md](03-background-workers.md) | Workers en segundo plano | Docker |
| [04-database.md](04-database.md) | Base de datos principal | PostgreSQL |
| [05-object-storage.md](05-object-storage.md) | Almacenamiento de archivos | Cloudinary |
| [06-auth-provider.md](06-auth-provider.md) | Proveedor de autenticación | Auth0 |
| [07-payment-provider.md](07-payment-provider.md) | Proveedor de pagos | NeoNet — Banco BI |
| [08-notification-system.md](08-notification-system.md) | Sistema de notificaciones | Amazon SES + WhatsApp Business API |
