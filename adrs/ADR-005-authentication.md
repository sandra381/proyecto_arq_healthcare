# ADR-005: Autenticación y Autorización

## Contexto

El sistema tiene cuatro tipos de usuarios con permisos completamente distintos: pacientes, médicos, recepcionistas y administradores de clínica. Un paciente puede agendar citas y ver su expediente, pero no puede ver los reportes de ingresos. Un médico puede escribir notas clínicas, pero no puede modificar el inventario. Es necesario un mecanismo que verifique la identidad de cada usuario y controle qué puede hacer dentro del sistema según su rol.

## Opciones Consideradas

### Opción 1: Sistema de autenticación propio

| Pros | Contras |
|---|---|
| Control total sobre el proceso de autenticación | Requiere implementar y mantener funcionalidades complejas: recuperación de contraseña, verificación de correo, bloqueo por intentos fallidos |
| Sin dependencia de servicios externos | La seguridad depende completamente del equipo, que no es especialista en seguridad |
| Sin costo adicional por usuario | Un error en la implementación puede comprometer todos los datos médicos del sistema |

### Opción 2: Proveedor de identidad administrado (Auth0)

| Pros | Contras |
|---|---|
| La seguridad es responsabilidad de especialistas | Costo mensual adicional por usuario activo |
| Funcionalidades listas: verificación de correo, recuperación de contraseña, autenticación en dos pasos | Dependencia de un servicio externo |
| El equipo se concentra en la lógica del negocio médico en vez de seguridad | Si el servicio cae, los usuarios no pueden iniciar sesión |
| Emite tokens estándar (JWT) que todos los módulos pueden verificar fácilmente | |
| Cumple con estándares de seguridad reconocidos internacionalmente | |

## Decisión

Se elige Auth0 como proveedor de identidad administrado.

Un sistema que maneja información médica sensible no puede permitirse errores de seguridad en la autenticación. Con un equipo de 6 a 10 ingenieros cuyo foco debe ser la lógica del negocio médico, implementar un sistema de autenticación propio sería un riesgo innecesario. Auth0 emite tokens JWT que todos los módulos del sistema pueden verificar de forma independiente sin consultar una base de datos centralizada, lo que encaja perfectamente con la arquitectura de módulos desacoplados que adoptamos.

**Roles y permisos del sistema:**

| Rol | Qué puede hacer |
|---|---|
| Paciente | Buscar médicos, agendar citas, pagar, ver su propio expediente, cancelar sus citas |
| Médico | Ver su agenda, escribir notas clínicas, emitir recetas, gestionar su disponibilidad |
| Recepcionista | Agendar y cancelar citas en nombre de pacientes, ver disponibilidad de médicos |
| Administrador | Ver reportes de ingresos, gestionar médicos, controlar inventario, suspender cuentas |

## Consecuencias

**Lo que habilita:**
- Los usuarios pueden iniciar sesión de forma segura sin que el equipo tenga que implementar mecanismos de seguridad complejos
- Cada módulo puede verificar la identidad del usuario de forma independiente usando el token JWT
- Es posible agregar autenticación en dos pasos sin modificar el código del sistema

**Riesgos y mitigación:**
- Si Auth0 tiene una interrupción, los usuarios no pueden iniciar sesión — se mitiga configurando un tiempo de vida del token razonablemente largo para que las sesiones activas continúen funcionando
- El costo aumenta con el número de usuarios — se mitiga eligiendo el plan que se ajuste al volumen de clínicas atendidas y revisándolo periódicamente

## ADRs Relacionados

- ADR-001 — Modelo de Despliegue
- ADR-006 — Diseño de API
