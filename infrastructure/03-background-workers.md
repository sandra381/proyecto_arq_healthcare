# 03 — Workers en Segundo Plano

## Responsabilidades

- Ejecutan las tareas automáticas del sistema en segundo plano — como enviar correos, actualizar reportes y liberar espacios expirados — sin que el usuario tenga que esperar a que terminen
- Escuchan los eventos publicados por la aplicación principal y ejecutan la acción correspondiente a cada uno, por ejemplo cuando se confirma una cita, el worker de notificaciones envía el correo al paciente y programa el recordatorio
- Reintentan automáticamente hasta 3 veces las tareas que no pudieron completarse antes de marcarlas como fallidas para revisión del administrador
- Operan de forma independiente entre sí, de modo que si uno falla, los demás continúan funcionando sin interrupción

## Por Qué Este Proyecto lo Necesita

El sistema tiene tareas que no deben bloquear al usuario pero que son críticas para el negocio. Por ejemplo, cuando un paciente confirma su cita, el sistema debe enviar un correo de confirmación, programar un recordatorio para 24 horas antes y actualizar los contadores del reporte diario. Si todas estas tareas corrieran dentro de la solicitud del usuario, el paciente tendría que esperar varios segundos a que todas terminen.

Los workers resuelven este problema ejecutando esas tareas en segundo plano. El paciente recibe su confirmación de inmediato y las tareas adicionales se procesan sin que él lo note.

## Elección de Tecnología

| Dimensión | Docker | Railway | AWS ECS |
|---|---|---|---|
| **Administrado o instalado** | Instalado por el equipo | Administrado por Railway | Administrado por AWS |
| **Complejidad operativa** | Baja — el equipo tiene control total | Muy baja — sin servidores que administrar | Media — requiere conocimiento de AWS |
| **Costo para clínica pequeña-mediana** | Gratuito — solo el costo del servidor | Desde $5 al mes con costo variable según uso | Costo variable según uso, sin techo fijo |
| **Ventaja principal** | Gratuito y consistente con el resto del sistema | Escala fácilmente sin gestionar infraestructura | Integración nativa con todos los servicios de AWS |

Se elige Docker porque es completamente gratuito, el equipo ya lo usa para la aplicación principal y cada worker puede reiniciarse de forma independiente sin afectar a los demás. Si en el futuro el sistema crece y el equipo prefiere no administrar servidores, la migración a Railway es natural — Railway soporta contenedores Docker directamente, por lo que no habría que cambiar el código, solo el lugar donde corre.

## Trade-offs

| Ventajas | Desventajas |
|---|---|
| Cada worker puede fallar y reiniciarse sin afectar al resto | Requiere más recursos de servidor que un proceso simple |
| Consistente con la tecnología del resto del sistema | El equipo debe monitorear el estado de cada worker |
| Fácil de escalar agregando más instancias de un worker específico | |
| Los logs de cada worker son independientes y fáciles de revisar | |

## Integración

Como se muestra en el diagrama de despliegue, los tres workers del sistema corren en la red interna junto a la aplicación principal. Cada worker está conectado a RabbitMQ para consumir eventos y a PostgreSQL para leer y escribir datos. El worker de notificaciones también se conecta al servicio externo de notificaciones para enviar correos y SMS. Si un worker falla, RabbitMQ retiene sus mensajes hasta que vuelva a estar disponible, sin perder ninguna tarea pendiente.
