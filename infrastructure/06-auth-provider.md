# 06 — Proveedor de Autenticación (Auth0)

## Responsabilidades

- Verifica la identidad de todos los usuarios del sistema — pacientes, médicos, recepcionistas y administradores — cuando inician sesión
- Emite tokens de acceso que los módulos del sistema usan para saber quién es el usuario y qué puede hacer
- Gestiona funcionalidades de seguridad como recuperación de contraseña, verificación de correo y bloqueo por intentos fallidos
- Controla que cada usuario solo pueda acceder a las funciones que corresponden a su rol

## Por Qué Este Proyecto lo Necesita

Un sistema que maneja información médica sensible no puede permitirse errores de seguridad en el proceso de autenticación. Implementar un sistema de login propio requeriría que el equipo maneje contraseñas cifradas, tokens de sesión, recuperación de cuentas y protección contra ataques — todo esto sin ser especialistas en seguridad.

El sistema tiene cuatro roles con permisos muy distintos: un paciente no puede ver los reportes de ingresos, un médico no puede modificar el inventario, una recepcionista no puede acceder a los expedientes clínicos. Auth0 maneja todo esto de forma segura sin que el equipo tenga que construirlo desde cero.

## Elección de Tecnología

| Dimensión | Auth0 | Firebase Authentication | Keycloak |
|---|---|---|---|
| **Administrado o instalado** | Administrado por Auth0 | Administrado por Google | Instalado por el equipo en su propio servidor |
| **Complejidad operativa** | Muy baja — Auth0 gestiona toda la seguridad | Baja — similar a Auth0, más orientado a apps móviles | Media — el equipo lo configura y mantiene, pero es muy potente |
| **Costo para clínica pequeña-mediana** | Plan gratuito hasta 25,000 usuarios activos — escala cambiando de plan sin modificar el código | Plan gratuito generoso — similar a Auth0 en costo y escalabilidad | Gratuito sin límite de usuarios — el costo es el tiempo del equipo para administrarlo |
| **Ventaja principal** | Especialistas en seguridad gestionan todo — fácil de escalar a largo plazo | Integración nativa con otros servicios de Google | Sin costo por usuario — ideal si el sistema crece mucho y el costo de Auth0 se vuelve alto |

Se elige Auth0 porque la seguridad de la autenticación es demasiado crítica para que la maneje un equipo sin experiencia especializada. Su plan gratuito cubre el volumen inicial de la clínica y escala sin necesidad de cambiar el código del sistema. Si en el futuro el número de usuarios crece considerablemente y el costo de Auth0 se vuelve alto, Keycloak sería la alternativa más económica — aunque requeriría que el equipo lo administre. Migrar entre proveedores de identidad es posible pero toma varios días de trabajo, por lo que es una decisión que conviene planificar con anticipación.

## Trade-offs

| Ventajas | Desventajas |
|---|---|
| La seguridad es responsabilidad de especialistas en Auth0 | Si Auth0 tiene una interrupción, los usuarios no pueden iniciar sesión |
| Plan gratuito cubre el volumen de la clínica | Introduce dependencia con un proveedor externo |
| El equipo se concentra en la lógica médica en vez de seguridad | El costo aumenta si el número de usuarios supera el plan gratuito |
| Tokens JWT verificables por cualquier módulo de forma independiente | |

## Integración

Como se muestra en el diagrama de despliegue, Auth0 es un servicio externo al que accede la aplicación principal para verificar la identidad del usuario en cada solicitud. Cuando un usuario inicia sesión, Auth0 emite un token que el sistema incluye en todas las solicitudes siguientes. Cada módulo verifica ese token de forma independiente para saber quién es el usuario y qué puede hacer, sin necesidad de consultar Auth0 en cada operación. Esto está justificado en el ADR-005.
