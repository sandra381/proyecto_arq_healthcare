# ADR-002: Estilo de Comunicación

## Contexto

El sistema tiene dos tipos de operaciones muy distintas. Por un lado, hay operaciones donde el usuario espera una respuesta inmediata — como buscar disponibilidad de un médico o reservar un espacio. Por otro lado, hay operaciones que pueden ocurrir en segundo plano sin que el usuario tenga que esperar — como enviar el correo de confirmación o actualizar los reportes de ingresos. Usar el mismo mecanismo de comunicación para ambas sería ineficiente y afectaría la experiencia del usuario.

## Opciones Consideradas

### Opción 1: Comunicación Síncrona para Todo

| Pros | Contras |
|---|---|
| Simple de implementar y depurar | El usuario debe esperar a que todas las operaciones terminen antes de recibir respuesta |
| El flujo es predecible y fácil de seguir | Si el envío del correo falla, la confirmación de la cita también falla |
| No requiere infraestructura adicional | Las operaciones lentas como notificaciones bloquean las operaciones rápidas |

### Opción 2: Comunicación Asíncrona para Todo

| Pros | Contras |
|---|---|
| Todas las operaciones son independientes entre sí | El usuario no sabe si su operación fue exitosa de inmediato |
| Ninguna operación lenta bloquea a las demás | Más difícil de depurar cuando algo falla |
| | Operaciones críticas como verificar disponibilidad no pueden ser asíncronas |

### Opción 3: Combinación — Síncrono para operaciones críticas, Asíncrono para efectos secundarios

| Pros | Contras |
|---|---|
| El usuario recibe respuesta inmediata en operaciones críticas | Requiere definir claramente cuándo usar cada estilo |
| Las operaciones secundarias no afectan la experiencia principal | Mayor complejidad que usar un solo estilo |
| Si una notificación falla, la cita ya está confirmada | |
| Cada módulo puede procesar sus eventos a su propio ritmo | |

## Decisión

Se elige la combinación de comunicación síncrona para operaciones críticas y asíncrona para efectos secundarios.

Las operaciones donde el usuario espera una respuesta — buscar disponibilidad, reservar un espacio, confirmar el pago — usan comunicación síncrona porque el resultado afecta directamente lo que el usuario ve en pantalla. Las operaciones que son consecuencia de esas acciones — enviar correos, actualizar reportes, descontar inventario — usan eventos asíncronos porque pueden ocurrir en segundo plano sin afectar la experiencia del usuario.

**Ejemplos concretos del sistema:**
- Síncrono: `GET /disponibilidad/:medicoId` — el paciente espera ver los horarios disponibles
- Síncrono: `POST /pagos/iniciar` — el paciente espera la confirmación del cobro
- Asíncrono: evento `PagoConfirmado` → módulo de Notificaciones envía el correo
- Asíncrono: evento `CitaCancelada` → módulo de Reportes actualiza los contadores del día

## Consecuencias

**Lo que habilita:**
- El usuario recibe respuesta inmediata en las operaciones que le importan
- Las notificaciones y actualizaciones secundarias no afectan el tiempo de respuesta principal
- Cada módulo puede procesar sus eventos de forma independiente sin bloquear a los demás

**Riesgos y mitigación:**
- Si un evento asíncrono falla, puede haber inconsistencias temporales — se mitiga con reintentos automáticos y registros de auditoría en cada módulo
- La depuración de errores en flujos asíncronos es más compleja — se mitiga con registros detallados de cada evento publicado y consumido

## ADRs Relacionados

- ADR-001 — Modelo de Despliegue
- ADR-004 — Bus de Eventos
