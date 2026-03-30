# ADRs — Registros de Decisiones Arquitectónicas

Esta carpeta contiene los registros de las decisiones de diseño más importantes del Healthcare Scheduling System. Cada ADR explica el problema que se estaba resolviendo, las opciones que se evaluaron y por qué se tomó la decisión final.

| Documento | Decisión | Estado |
|---|---|---|
| [ADR-001-deployment-model.md](ADR-001-deployment-model.md) | Monolito Modular con Event Driven elegido sobre Microservicios. | Aceptado |
| [ADR-002-communication-style.md](ADR-002-communication-style.md) | Comunicación síncrona para operaciones críticas y asíncrona para efectos secundarios. | Aceptado |
| [ADR-003-database-strategy.md](ADR-003-database-strategy.md) | Una base de datos PostgreSQL compartida con espacios separados por módulo. | Aceptado |
| [ADR-004-event-bus.md](ADR-004-event-bus.md) | RabbitMQ elegido como bus de eventos sobre Kafka y colas en la nube. | Aceptado |
| [ADR-005-authentication.md](ADR-005-authentication.md) | Auth0 como proveedor de identidad con 4 roles definidos. | Aceptado |
| [ADR-006-api-design.md](ADR-006-api-design.md) | REST para todas las APIs externas del sistema. | Aceptado |
| [ADR-007-frontend-architecture.md](ADR-007-frontend-architecture.md) | Aplicación híbrida con React adaptada por rol de usuario. | Aceptado |
