# Healthcare Scheduling System

Sistema de agendamiento médico diseñado para clínicas pequeñas y medianas en Guatemala. Permite a los pacientes buscar médicos, agendar citas y pagar en línea, mientras que el personal médico gestiona su agenda, expedientes clínicos e inventario desde una sola plataforma.

## Arquitecturas Comparadas

| Arquitectura | Descripción |
|---|---|
| **Monolito Modular con Event Driven** | Todo el sistema en una sola aplicación con módulos desacoplados que se comunican por eventos — la arquitectura elegida. Ver [propuesta completa](proposal/01-high-level-architecture.md) |
| **Microservicios** | Cada módulo como un servicio independiente desplegado por separado. Ver [comparación detallada](proposal/01-high-level-architecture.md) |

## Arquitectura Elegida

Se eligió el **Monolito Modular con Event Driven** porque permite mantener la integridad de los datos médicos con transacciones atómicas, reduce la complejidad operativa para un equipo de 6 a 10 ingenieros y mantiene el costo de infraestructura bajo para el volumen de clínicas pequeñas y medianas. Ver [justificación completa](adrs/ADR-001-deployment-model.md).

## Navegación del Repositorio

### Carpetas principales

| Carpeta | Descripción |
|---|---|
| [proposals/](proposal/) | Propuestas de arquitectura — decisiones de alto nivel sobre la estructura del sistema. |
| [adrs/](adrs/) | Registros de decisiones arquitectónicas — justificación de cada decisión técnica importante. |
| [diagrams/](diagrams/) | Diagramas de arquitectura — representaciones visuales del sistema. |
| [infrastructure/](infrastructure/) | Componentes de infraestructura — tecnologías usadas y por qué se eligieron. |

---

### Proposals

| Documento | Descripción |
|---|---|
| [01-high-level-architecture.md](proposal/01-high-level-architecture.md) | Comparación entre Monolito Modular con Event Driven y Microservicios con justificación de la elección. |
| [02-bounded-contexts.md](proposal/02-bounded-contexts.md) | Los 9 contextos delimitados del sistema con sus entidades, eventos y patrones de integración. |
| [03-service-module-decomposition.md](proposal/03-service-module-decomposition.md) | Estructura de carpetas del código, responsabilidades por módulo y procesos separados. |
| [04-data-flow-and-interactions.md](proposal/04-data-flow-and-interactions.md) | Los 3 flujos principales del sistema con diagramas de secuencia y caminos de fallo. |

---

### ADRs

| Documento | Decisión |
|---|---|
| [ADR-001-deployment-model.md](adrs/ADR-001-deployment-model.md) | Monolito Modular con Event Driven elegido sobre Microservicios. |
| [ADR-002-communication-style.md](adrs/ADR-002-communication-style.md) | Comunicación síncrona para operaciones críticas y asíncrona para efectos secundarios. |
| [ADR-003-database-strategy.md](adrs/ADR-003-database-strategy.md) | Una base de datos PostgreSQL compartida con espacios separados por módulo. |
| [ADR-004-event-bus.md](adrs/ADR-004-event-bus.md) | RabbitMQ elegido como bus de eventos sobre Kafka y colas en la nube. |
| [ADR-005-authentication.md](adrs/ADR-005-authentication.md) | Auth0 como proveedor de identidad con 4 roles definidos. |
| [ADR-006-api-design.md](adrs/ADR-006-api-design.md) | REST para todas las APIs externas del sistema. |
| [ADR-007-frontend-architecture.md](adrs/ADR-007-frontend-architecture.md) | Aplicación híbrida con React adaptada por rol de usuario. |

---

### Diagrams

| Documento | Descripción |
|---|---|
| [01-system-context.md](diagrams/01-system-context.md) | C4 Nivel 1 — el sistema como una sola caja rodeada de actores y sistemas externos. |
| [02-bounded-context-map.md](diagrams/02-bounded-context-map.md) | Mapa DDD con los 9 contextos agrupados por tipo de dominio y patrones de integración. |
| [03-data-flow.md](diagrams/03-data-flow.md) | 5 diagramas de secuencia con los flujos principales, caminos felices y caminos de fallo. |
| [04-deployment.md](diagrams/04-deployment.md) | Topología de despliegue — contenedores, base de datos, workers y servicios externos. |

---

### Infrastructure

| Documento | Componente | Tecnología |
|---|---|---|
| [01-load-balancer.md](infrastructure/01-load-balancer.md) | Balanceador de carga | AWS ALB |
| [02-message-queue.md](infrastructure/02-message-queue.md) | Cola de mensajes | RabbitMQ |
| [03-background-workers.md](infrastructure/03-background-workers.md) | Workers en segundo plano | Docker |
| [04-database.md](infrastructure/04-database.md) | Base de datos principal | PostgreSQL |
| [05-object-storage.md](infrastructure/05-object-storage.md) | Almacenamiento de archivos | Cloudinary |
| [06-auth-provider.md](infrastructure/06-auth-provider.md) | Proveedor de autenticación | Auth0 |
| [07-payment-provider.md](infrastructure/07-payment-provider.md) | Proveedor de pagos | NeoNet — Banco BI |
| [08-notification-system.md](infrastructure/08-notification-system.md) | Sistema de notificaciones | Amazon SES + WhatsApp Business API |
