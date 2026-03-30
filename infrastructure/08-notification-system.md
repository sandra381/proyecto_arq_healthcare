# 08 — Sistema de Notificaciones (Amazon SES + WhatsApp Business API)

## Responsabilidades

- Envía correos electrónicos de confirmación, recordatorio y cancelación de citas a pacientes y médicos a través de Amazon SES
- Envía mensajes de WhatsApp a los pacientes 24 horas antes de su cita para reducir las inasistencias
- Reintenta automáticamente el envío de mensajes que fallan hasta 3 veces antes de marcarlos como fallidos
- Registra el comprobante de cada mensaje enviado exitosamente para auditoría y seguimiento

## Por Qué Este Proyecto lo Necesita

Las inasistencias a citas médicas representan tiempo perdido para el médico y un espacio que pudo haberse dado a otro paciente. Los recordatorios automáticos 24 horas antes reducen significativamente este problema sin que el personal de la clínica tenga que llamar a cada paciente manualmente.

Además, el paciente necesita recibir confirmación inmediata cuando su cita es registrada. Sin notificaciones automáticas, el paciente podría dudar si su cita quedó confirmada o no, lo que genera llamadas innecesarias a la clínica y carga de trabajo extra para la recepcionista.

Se eligen dos canales de comunicación porque en Guatemala los pacientes revisan el correo para documentos formales como confirmaciones y comprobantes, pero responden mejor al WhatsApp para recordatorios rápidos del día a día.

## Elección de Tecnología

| Dimensión | Amazon SES + WhatsApp Business API | Brevo + Twilio | SendGrid + Twilio |
|---|---|---|---|
| **Administrado o instalado** | Administrado por AWS y Meta respectivamente | Administrado por Brevo y Twilio | Administrado por Twilio |
| **Complejidad operativa** | Media — requiere configuración inicial de dominio en AWS y verificación de cuenta en Meta | Baja — ambas plataformas tienen interfaces simples | Baja — bien documentado pero SendGrid ya no tiene plan gratuito permanente |
| **Costo para clínica pequeña-mediana** | Amazon SES: muy bajo por correo. WhatsApp: costo por conversación, primeras 1,000 conversaciones gratuitas al mes | Brevo: plan gratuito con 300 correos al día. Twilio SMS: costo variable por país, puede ser elevado en Guatemala | SendGrid: sin plan gratuito permanente desde mayo 2025. Twilio SMS: costo variable por país |
| **Ventaja principal** | Amazon SES es el servicio de correo más confiable del mundo. WhatsApp es el canal de mensajería más usado en Guatemala | Brevo tiene plan gratuito generoso. Opción más simple de configurar | Ampliamente conocido pero con limitaciones de costo para clínicas pequeñas |

Se elige Amazon SES para correos y WhatsApp Business API para mensajes porque Amazon SES es uno de los servicios de correo más confiables del mercado y funciona sin restricciones en Guatemala. WhatsApp Business API es la opción más efectiva para mensajes en Guatemala donde la mayoría de usuarios prefiere WhatsApp sobre el SMS tradicional. Esta combinación es económica, confiable y perfectamente adaptada al contexto guatemalteco.

## Trade-offs

| Ventajas | Desventajas |
|---|---|
| Amazon SES es extremadamente confiable y de bajo costo | La configuración inicial de Amazon SES requiere verificar el dominio y configurar registros DNS — toma unas horas la primera vez |
| WhatsApp es el canal preferido en Guatemala — mayor probabilidad de que el paciente vea el mensaje | WhatsApp Business API requiere aprobación de Meta antes de poder usarlo — el proceso puede tomar varios días |
| Si en el futuro el sistema migra a AWS, todo estaría en el mismo ecosistema | Dependencia de dos proveedores externos en vez de uno |
| Las primeras 1,000 conversaciones de WhatsApp al mes son gratuitas — suficiente para empezar | Si WhatsApp tiene una interrupción, ese canal de notificaciones deja de funcionar temporalmente |

## Integración

Como se muestra en el diagrama de despliegue (`diagrams/04-deployment.md`), el sistema de notificaciones es accedido por el worker de notificaciones que corre en un contenedor separado. Cuando ocurre un evento importante — una cita confirmada, un pago fallido, una cita cancelada — el módulo correspondiente publica el evento en RabbitMQ, el worker lo consume y decide qué canal usar: Amazon SES para correos formales de confirmación y comprobantes, y WhatsApp Business API para recordatorios rápidos al paciente. El módulo de Notificaciones guarda en PostgreSQL el comprobante de cada mensaje enviado para que el sistema pueda verificar que el paciente recibió la información.
