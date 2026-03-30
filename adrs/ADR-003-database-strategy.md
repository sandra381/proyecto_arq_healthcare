# ADR-003: Estrategia de Base de Datos

## Contexto

El sistema tiene 9 módulos, cada uno responsable de una parte distinta del negocio. Es necesario decidir cómo organizar el almacenamiento de datos para que los módulos sean independientes entre sí pero sin multiplicar la infraestructura innecesariamente. La información médica es sensible y requiere garantías de consistencia — por ejemplo, cuando un paciente agenda una cita, el sistema debe crear el registro y bloquear el espacio del médico al mismo tiempo — si algo falla, el sistema cancela todo y vuelve a como estaba antes.

## Opciones Consideradas

### Opción 1: Una base de datos compartida sin separación

| Pros | Contras |
|---|---|
| Simple de administrar | Cualquier módulo puede acceder a los datos de otro, rompiendo el aislamiento |
| Bajo costo operativo | Los cambios en una tabla pueden afectar a múltiples módulos sin que sea evidente |
| Transacciones entre módulos son naturales | Con el tiempo se convierte en un sistema difícil de mantener |

### Opción 2: Una base de datos por módulo

| Pros | Contras |
|---|---|
| Aislamiento total entre módulos | 9 bases de datos que administrar, respaldar y monitorear |
| Cada módulo puede usar la tecnología más adecuada | Las transacciones entre módulos son imposibles sin mecanismos de compensación |
| | Costo de infraestructura significativamente mayor |
| | Equipo de 6 a 10 ingenieros no puede operar 9 bases de datos de forma responsable |

### Opción 3: Una base de datos compartida con espacios separados por módulo

| Pros | Contras |
|---|---|
| Un solo sistema de base de datos que administrar | Requiere disciplina para respetar los límites entre espacios |
| Cada módulo opera exclusivamente en su propio espacio | No permite usar tecnologías distintas por módulo |
| Las transacciones críticas pueden abarcar múltiples módulos cuando es necesario | |
| Bajo costo operativo con buen aislamiento lógico | |

## Decisión

Se elige una base de datos PostgreSQL compartida con espacios separados por módulo.

Con un equipo de 6 a 10 ingenieros, administrar 9 bases de datos independientes no es viable. Al mismo tiempo, una base de datos completamente compartida sin separación rompería el aislamiento entre módulos. La solución de espacios separados dentro de una misma base de datos permite que cada módulo opere de forma independiente a nivel de código, mientras se mantiene la posibilidad de hacer transacciones atómicas cuando es necesario — como al crear una cita y bloquear el espacio del médico al mismo tiempo.

## Consecuencias

**Lo que habilita:**
- Un solo sistema de base de datos que respaldar, monitorear y mantener
- Aislamiento lógico entre módulos sin multiplicar la infraestructura
- Transacciones atómicas para operaciones críticas como el agendamiento de citas

**Riesgos y mitigación:**
- Si un módulo accede al espacio de otro directamente, el aislamiento se rompe — se mitiga con reglas de acceso a nivel de código verificadas automáticamente en cada cambio
- A medida que el sistema crece, una sola base de datos puede convertirse en un cuello de botella — se mitiga con índices bien diseñados y la posibilidad de extraer módulos con sus propios espacios en el futuro

## ADRs Relacionados

- ADR-001 — Modelo de Despliegue
- ADR-002 — Estilo de Comunicación
