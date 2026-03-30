# ADR-004: Bus de Eventos

## Contexto

El sistema usa comunicación por eventos para que los módulos reaccionen a lo que ocurre en otros módulos sin llamarse directamente. Por ejemplo, cuando se confirma un pago, los módulos de Citas, Horarios y Notificaciones deben enterarse y reaccionar cada uno por su cuenta. Se necesita una herramienta que transporte estos mensajes de forma confiable entre los módulos y los workers que corren en procesos separados.

## Opciones Consideradas

### Opción 1: RabbitMQ

| Pros | Contras |
|---|---|
| Diseñado específicamente para enrutamiento de mensajes entre módulos | No está diseñado para almacenar grandes volúmenes de eventos históricos |
| Fácil de configurar y operar con un equipo pequeño | Menos adecuado si en el futuro se necesita procesar millones de eventos por segundo |
| Soporte nativo para reintentos automáticos y colas de mensajes fallidos | |
| Bajo consumo de recursos para el volumen esperado del sistema | |
| Amplia documentación y comunidad activa | |

### Opción 2: Apache Kafka

| Pros | Contras |
|---|---|
| Diseñado para procesar millones de eventos por segundo | Complejo de configurar y operar, especialmente para equipos pequeños |
| Almacena el historial completo de eventos | Consume muchos más recursos que RabbitMQ para el volumen del sistema |
| Ideal para sistemas de análisis en tiempo real | Su complejidad no está justificada para clínicas pequeñas y medianas |

### Opción 3: Cola administrada en la nube (AWS SQS)

| Pros | Contras |
|---|---|
| Sin necesidad de administrar infraestructura propia | Costo variable que aumenta con el volumen de mensajes |
| Alta disponibilidad garantizada por el proveedor | Introduce dependencia directa con un proveedor específico |
| Fácil de escalar | Menor control sobre el comportamiento de la cola |

## Decisión

Se elige RabbitMQ como bus de eventos del sistema.

El volumen de mensajes esperado para clínicas pequeñas y medianas — confirmaciones de pago, notificaciones, actualizaciones de disponibilidad — es completamente manejable con RabbitMQ. La complejidad operativa de Kafka no está justificada para este volumen, y la dependencia con un proveedor de nube específica como AWS SQS limitaría la portabilidad del sistema. RabbitMQ ofrece exactamente lo que el sistema necesita: enrutamiento confiable de mensajes, reintentos automáticos y una curva de aprendizaje razonable para un equipo de 6 a 10 ingenieros.

## Consecuencias

**Lo que habilita:**
- Los módulos pueden comunicarse de forma asíncrona sin conocerse entre sí
- Los mensajes que fallan se reintentarán automáticamente sin intervención manual
- Los workers de tareas automáticas pueden correr en procesos separados sin afectar el proceso principal

**Riesgos y mitigación:**
- RabbitMQ requiere ser instalado y administrado por el equipo — se mitiga usando una imagen Docker oficial en un contenedor administrado
- Si el volumen de mensajes crece significativamente, RabbitMQ podría quedarse corto — se mitiga monitoreando el tamaño de las colas y planificando la migración a Kafka si fuera necesario

## ADRs Relacionados

- ADR-001 — Modelo de Despliegue
- ADR-002 — Estilo de Comunicación
