# 01 — Balanceador de Carga

## Responsabilidades

- Recibe todo el tráfico de los usuarios (pacientes, médicos, recepcionistas y administradores) y lo distribuye entre las instancias disponibles de la aplicación principal
- Verifica constantemente que las instancias de la aplicación estén funcionando y deja de enviarles tráfico si detecta que alguna falló
- Termina las conexiones HTTPS de los usuarios para que la aplicación interna solo maneje tráfico HTTP sin cifrado
- Permite agregar o quitar instancias de la aplicación sin interrumpir el servicio a los usuarios

## Por Qué Este Proyecto lo Necesita

Sin un balanceador de carga, todo el tráfico llegaría directamente a una sola instancia de la aplicación. Si esa instancia falla, el sistema completo deja de funcionar y ningún paciente puede agendar citas. En una clínica, esto significa que los pacientes que llaman para confirmar una cita no pueden hacerlo y el médico no puede ver su agenda del día.

Además, en los momentos de mayor tráfico — cuando varios usuarios buscan citas al mismo tiempo — una sola instancia podría sobrecargarse y responder lentamente. El balanceador de carga distribuye las solicitudes entre varias instancias para que todos los usuarios reciban una respuesta rápida.

## Elección de Tecnología

| Dimensión | AWS ALB | Nginx | Cloudflare |
|---|---|---|---|
| **Administrado o instalado** | Administrado por AWS | Instalado y configurado por el equipo | Administrado por Cloudflare |
| **Complejidad operativa** | Baja — AWS lo gestiona automáticamente | Media — requiere configuración y mantenimiento manual | Muy baja — interfaz simple sin configuración compleja |
| **Costo para clínica pequeña-mediana** | Se paga por uso — sin costo fijo alto, ideal para volúmenes de tráfico moderados | Sin costo de software, pero requiere pagar un servidor propio y tiempo del equipo para configurarlo y mantenerlo | Tiene un plan gratuito que cubre el volumen básico de una clínica, con opciones de pago si el sistema crece |
| **Ventaja principal** | Se integra nativamente con otros servicios de AWS y escala automáticamente | Muy flexible y ampliamente conocido por los desarrolladores | Fácil de usar y además protege contra ataques externos |

Se elige AWS ALB porque al ser administrado por AWS no requiere que el equipo lo configure ni lo mantenga. Para un equipo de 6 a 10 ingenieros, eliminar esa carga operativa es una ventaja significativa.

## Trade-offs

| Ventajas | Desventajas |
|---|---|
| No requiere configuración ni mantenimiento por parte del equipo | Introduce dependencia con el proveedor de nube AWS |
| Escala automáticamente según el volumen de tráfico | Tiene un costo mensual aunque el tráfico sea bajo |
| Se integra nativamente con los demás servicios de AWS | Menos control sobre la configuración avanzada |
| Alta disponibilidad garantizada por AWS | |

## Integración

El balanceador de carga es el primer punto de entrada del sistema. Recibe las solicitudes de los usuarios desde la red pública y las redirige a la aplicación principal en la red interna. Como se muestra en el diagrama de despliegue, ningún usuario accede directamente a la aplicación — todo pasa primero por el balanceador. Si en el futuro se necesitan más instancias de la aplicación, el balanceador las detecta automáticamente y distribuye el tráfico entre todas.
