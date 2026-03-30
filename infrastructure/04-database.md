# 04 — Base de Datos Principal (PostgreSQL)

## Responsabilidades

- Almacena toda la información estructurada del sistema: pacientes, médicos, citas, pagos, expedientes y reportes
- Garantiza que operaciones críticas como agendar una cita y bloquear el espacio del médico ocurran juntas o no ocurran ninguna
- Mantiene los datos de cada módulo en su propio espacio separado, evitando que un módulo acceda a los datos de otro
- Gestiona múltiples solicitudes simultáneas sin que los datos queden en un estado inconsistente

## Por Qué Este Proyecto lo Necesita

Un sistema de agendamiento médico maneja información sensible y operaciones que no pueden quedar a medias. Cuando un paciente agenda una cita, el sistema debe crear el registro de la cita y bloquear el espacio del médico al mismo tiempo — si algo falla, el sistema cancela todo y vuelve a como estaba antes. Esto requiere una base de datos que soporte transacciones confiables.

Además, el sistema tiene 9 módulos con datos muy distintos entre sí — citas, pagos, expedientes, inventario — y es necesario que cada módulo opere de forma aislada sin que los cambios de uno afecten a los demás. PostgreSQL permite crear espacios separados dentro de la misma base de datos, lo que mantiene ese aislamiento sin necesitar múltiples bases de datos independientes.

## Elección de Tecnología

| Dimensión | PostgreSQL | MySQL | MongoDB |
|---|---|---|---|
| **Administrado o instalado** | Instalado por el equipo o administrado en la nube | Instalado por el equipo o administrado en la nube | Instalado por el equipo o administrado en la nube |
| **Complejidad operativa** | Baja — ampliamente documentado y con herramientas maduras | Baja — similar a PostgreSQL | Media — requiere diseño diferente al relacional |
| **Costo para clínica pequeña-mediana** | Gratuito — sin costo de licencia | Gratuito en su versión comunitaria | Gratuito en su versión comunitaria |
| **Ventaja principal** | Transacciones robustas y soporte nativo para esquemas separados | Simple y ampliamente conocido | Flexible para datos no estructurados |

Se elige **PostgreSQL** porque sus transacciones son más robustas que las de MySQL en operaciones complejas que afectan múltiples tablas al mismo tiempo, lo cual es crítico para el agendamiento de citas. MongoDB queda descartado porque el sistema maneja datos altamente estructurados y relacionados entre sí, que es exactamente para lo que están diseñadas las bases de datos relacionales.

## Trade-offs

| Ventajas | Desventajas |
|---|---|
| Transacciones confiables para operaciones críticas como el agendamiento | Requiere ser instalado y mantenido por el equipo |
| Soporte nativo para esquemas separados por módulo | Escalar horizontalmente es más complejo que con bases de datos NoSQL |
| Gratuito sin costo de licencia | Si la base de datos falla, todo el sistema se ve afectado |
| Ampliamente conocido por cualquier ingeniero del equipo | |

## Integración

Como se muestra en el diagrama de despliegue, PostgreSQL se encuentra en la red interna y es accedida tanto por la aplicación principal como por los tres workers. Cada componente opera únicamente en su propio espacio de la base de datos — la aplicación no puede leer directamente los datos del worker de reportes ni viceversa. Esta separación está definida en el ADR-003 y se refuerza mediante reglas de acceso en el código del sistema.
