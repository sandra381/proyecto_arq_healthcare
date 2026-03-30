# ADR-007: Arquitectura de Frontend

## Contexto

El sistema tiene cuatro tipos de usuarios con necesidades muy distintas. Los pacientes necesitan buscar médicos y agendar citas principalmente desde su teléfono. Los médicos necesitan ver su agenda y escribir notas clínicas desde una computadora en el consultorio. Los recepcionistas gestionan citas durante todo el día desde una pantalla fija. Los administradores consultan reportes de ingresos ocasionalmente desde cualquier dispositivo. Es necesario decidir cómo construir la parte visual del sistema que cada uno de estos usuarios ve.

## Opciones Consideradas

### Opción 1: Aplicación de una sola página (SPA) con React

| Pros | Contras |
|---|---|
| Experiencia fluida sin recargas de página | La primera carga puede ser lenta si la aplicación es muy grande |
| Un solo código base para todos los roles | El navegador hace más trabajo, lo que puede afectar dispositivos de gama baja |
| El equipo trabaja en una sola tecnología | Los motores de búsqueda no indexan bien el contenido generado en el cliente |
| Fácil de adaptar la interfaz según el rol del usuario | |

### Opción 2: Aplicaciones separadas por rol

| Pros | Contras |
|---|---|
| Cada aplicación está optimizada para las necesidades de su rol | Múltiples repositorios y pipelines de despliegue que mantener |
| Los cambios en una aplicación no afectan a las demás | El equipo debe mantener código duplicado entre aplicaciones |
| | Mayor tiempo de desarrollo inicial |

### Opción 3: Aplicación híbrida — React con renderizado en servidor para las páginas públicas

| Pros | Contras |
|---|---|
| Las páginas públicas como la búsqueda de médicos cargan más rápido | Mayor complejidad de configuración inicial |
| Mejor experiencia en dispositivos con conexión lenta | El equipo debe entender tanto el renderizado en servidor como en cliente |
| Las páginas de administración siguen siendo dinámicas y fluidas | |
| Un solo código base para todos los roles | |

## Decisión

Se elige la aplicación híbrida con React — renderizado en servidor para las páginas públicas y renderizado en cliente para los paneles de cada rol.

Las páginas que los pacientes ven antes de iniciar sesión — búsqueda de médicos, especialidades disponibles — deben cargar rápido en cualquier dispositivo y conexión, lo que favorece el renderizado en servidor. Una vez que el usuario inicia sesión, la experiencia debe ser fluida e interactiva, lo que favorece el renderizado en cliente. Un solo código base con interfaces adaptadas por rol permite que el equipo de 6 a 10 ingenieros mantenga todo sin duplicar esfuerzo.

**Interfaces por rol:**
- **Paciente** — optimizada para móvil, enfocada en búsqueda y agendamiento
- **Médico** — optimizada para escritorio, con agenda diaria y acceso a notas clínicas
- **Recepcionista** — panel de gestión de citas del día con vista de disponibilidad en tiempo real
- **Administrador** — tablero de reportes de ingresos y gestión de médicos e inventario

## Consecuencias

**Lo que habilita:**
- Los pacientes tienen una experiencia rápida desde su teléfono incluso con conexión lenta
- El equipo mantiene un solo código base en vez de múltiples aplicaciones separadas
- La interfaz se adapta automáticamente según el rol del usuario que inicia sesión

**Riesgos y mitigación:**
- La configuración inicial del renderizado híbrido es más compleja que una SPA simple — se mitiga usando un framework con soporte nativo para este patrón como Next.js
- Si la aplicación crece mucho, el tiempo de carga inicial puede aumentar — se mitiga dividiendo el código en partes que se cargan solo cuando se necesitan

## ADRs Relacionados

- ADR-005 — Autenticación y Autorización
- ADR-006 — Diseño de API
