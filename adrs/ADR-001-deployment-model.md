# ADR-001: Modelo de Despliegue
## Contexto

El Healthcare Scheduling System necesita una arquitectura de despliegue que soporte a pacientes, médicos, recepcionistas y administradores de clínica. El equipo de desarrollo es de 6 a 10 ingenieros y el sistema está dirigido a clínicas pequeñas y medianas con un volumen de tráfico moderado. La principal preocupación es mantener la integridad de los datos médicos — Cuando un paciente agenda una cita, el sistema tiene que crear el registro y bloquear el espacio del médico al mismo tiempo. Si algo sale mal en el proceso, ninguna de las dos cosas debe quedar hecha a medias.

## Opciones Consideradas

### Opción 1: Monolito Modular con Event Driven

| Pros | Contras |
|---|---|
| Un solo proceso que desplegar y monitorear | Si hay un error grave puede afectar toda la aplicación |
| Todo el equipo trabaja en un solo repositorio | Requiere disciplina para mantener los límites entre módulos |
| Las transacciones de base de datos son atómicas por naturaleza | Escalar partes específicas del sistema no es posible de forma independiente |
| Los módulos se comunican por eventos internos sin latencia de red | El rendimiento de todos los módulos se ve afectado si uno consume demasiados recursos |
| Costo de infraestructura bajo desde el primer día | |

### Opción 2: Microservicios

| Pros | Contras |
|---|---|
| Cada servicio puede escalarse de forma independiente | Requiere coordinar 9 servicios independientes desde el primer día |
| Un servicio caído no afecta a los demás | Las transacciones entre servicios requieren mecanismos de compensación complejos |
| Cada servicio puede actualizarse sin tocar los demás | El equipo de 6 a 10 ingenieros no tiene capacidad para mantener 9 pipelines independientes |
| Tecnología independiente por servicio | El costo de infraestructura se multiplica desde el inicio |
| | La latencia de red entre servicios afecta el tiempo de respuesta |

## Decisión

Se elige el Monolito Modular con Event Driven.

Con un equipo de 6 a 10 ingenieros y un volumen de usuarios propio de clínicas pequeñas y medianas, los microservicios introducen una complejidad operativa que no está justificada. La necesidad de que el agendamiento de citas y el descuento de disponibilidad ocurran de forma atómica favorece directamente al monolito, donde esto se resuelve con una sola transacción de base de datos. Los módulos se mantienen desacoplados gracias a la comunicación por eventos internos, lo que permite crecer ordenadamente sin los costos de los microservicios.

## Consecuencias

**Lo que habilita:**
- El equipo puede desarrollar y desplegar el sistema completo sin coordinación entre múltiples pipelines
- Las transacciones de base de datos garantizan consistencia en operaciones críticas como el agendamiento
- El costo de infraestructura se mantiene bajo durante las etapas iniciales del proyecto

**Riesgos y mitigación:**
- Si los desarrolladores mezclan el código de un módulo con el de otro, el sistema se vuelve difícil de mantener — se mitiga con reglas de importación automáticas verificadas en cada cambio
- Si el sistema crece a hospitales con miles de citas diarias, el monolito podría quedarse corto — se mitiga diseñando los módulos con límites claros que permitan extraerlos como servicios independientes en el futuro

## ADRs Relacionados

- ADR-002 — Estilo de Comunicación
- ADR-003 — Estrategia de Base de Datos
- ADR-004 — Bus de Eventos
