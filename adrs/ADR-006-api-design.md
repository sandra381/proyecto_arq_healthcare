# ADR-006: Diseño de API

## Contexto

El sistema necesita una forma de comunicarse con las aplicaciones que usan los pacientes, médicos, recepcionistas y administradores. Estas aplicaciones — ya sea una app móvil o una página web — deben poder buscar disponibilidad, agendar citas, procesar pagos y consultar expedientes. La API debe ser fácil de consumir para los desarrolladores del frontend y lo suficientemente estable para no romper las aplicaciones con cada cambio.

## Opciones Consideradas

### Opción 1: REST

| Pros | Contras |
|---|---|
| Ampliamente conocido por todos los desarrolladores | Puede requerir múltiples llamadas para obtener datos relacionados |
| Fácil de documentar y probar | No tiene un esquema de datos estricto por defecto |
| Compatible con cualquier cliente: web, móvil, herramientas de terceros | |
| Los errores HTTP son intuitivos y estándar | |
| No requiere herramientas adicionales para consumirlo | |

### Opción 2: GraphQL

| Pros | Contras |
|---|---|
| El cliente puede pedir exactamente los datos que necesita | Curva de aprendizaje mayor para el equipo |
| Reduce el número de llamadas al servidor | Más complejo de implementar y depurar |
| Un solo punto de entrada para todas las consultas | El caché es más difícil de implementar que con REST |
| | Puede ser excesivo para un sistema con operaciones bien definidas |

### Opción 3: gRPC

| Pros | Contras |
|---|---|
| Muy rápido para comunicación entre servicios internos | No es compatible de forma nativa con navegadores web |
| Esquema de datos estricto definido desde el inicio | Requiere herramientas específicas para probarlo |
| | Difícil de consumir desde aplicaciones móviles sin capas adicionales |

## Decisión

Se elige REST para todas las APIs externas del sistema.

Las aplicaciones de pacientes, médicos, recepcionistas y administradores son los principales consumidores de la API. REST es el estilo más conocido por los desarrolladores frontend y es compatible de forma nativa con navegadores y aplicaciones móviles sin herramientas adicionales. Las operaciones del sistema están bien definidas — buscar disponibilidad, agendar cita, procesar pago — lo que hace que las ventajas de GraphQL no justifiquen su complejidad adicional. gRPC queda descartado porque las aplicaciones web no pueden consumirlo directamente.

**Ejemplos de endpoints del sistema:**
- `GET /medicos?especialidad=cardiologia` — buscar médicos por especialidad
- `GET /disponibilidad/:medicoId?fecha=2025-06-10` — consultar horarios disponibles
- `POST /citas` — crear una nueva cita
- `PUT /citas/:id/cancelar` — cancelar una cita existente
- `GET /expediente/:pacienteId` — consultar el expediente de un paciente

## Consecuencias

**Lo que habilita:**
- Cualquier desarrollador del equipo puede consumir y entender la API sin capacitación adicional
- Las aplicaciones móviles y web pueden comunicarse con el sistema usando herramientas estándar
- La API puede documentarse de forma clara usando herramientas como Swagger

**Riesgos y mitigación:**
- Algunos flujos pueden requerir múltiples llamadas al servidor — se mitiga diseñando endpoints que devuelvan la información necesaria para cada pantalla sin requerir llamadas adicionales
- Los cambios en la API pueden romper las aplicaciones existentes — se mitiga versionando la API desde el inicio

## ADRs Relacionados

- ADR-005 — Autenticación y Autorización
- ADR-007 — Arquitectura de Frontend
