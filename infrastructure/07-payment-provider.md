# 07 — Proveedor de Pagos (NeoNet — Banco BI)

## Responsabilidades

- Procesa los cobros de las consultas médicas cuando un paciente confirma su cita
- Gestiona los reembolsos cuando una cita es cancelada con derecho a devolución
- Almacena de forma segura los métodos de pago de los pacientes para que no tengan que ingresar sus datos en cada consulta
- Notifica al sistema a través de su API cuando un pago es confirmado, rechazado o reembolsado

## Por Qué Este Proyecto lo Necesita

El sistema nunca debe manejar datos de tarjetas de crédito directamente — hacerlo requeriría cumplir con estándares de seguridad muy exigentes y costosos que están fuera del alcance de un equipo pequeño. NeoNet actúa como intermediario: el paciente ingresa sus datos de tarjeta directamente en la interfaz segura de NeoNet, respaldada por Banco BI, y el sistema solo recibe la confirmación o el rechazo del cobro.

En una clínica, los pagos fallidos deben manejarse de forma inmediata — si el cobro de una consulta no se procesa, la cita no debe confirmarse y el espacio debe liberarse para otro paciente. NeoNet notifica al sistema de estos eventos a través de su API, lo que permite que el sistema reaccione automáticamente.

## Elección de Tecnología

| Dimensión | NeoNet — Banco BI (Pay Bi) | BAC Credomatic | Recurrente |
|---|---|---|---|
| **Administrado o instalado** | Administrado por Banco BI | Administrado por BAC Credomatic | Administrado por Recurrente |
| **Complejidad operativa** | Media — requiere afiliación formal pero tiene API bien documentada | Media — proceso de integración más tradicional | Baja — afiliación 100% en línea sin trámites complejos |
| **Costo para clínica pequeña-mediana** | Comisión por transacción negociable según volumen — sin costo fijo inicial | Comisión por transacción — tarifas pueden ser elevadas para volúmenes bajos | 4.5% por transacción sin costos fijos — si no hay ventas no hay cobro |
| **Ventaja principal** | Integración vía API con respaldo de Cybersource y Visa — ideal para sistemas digitales formales | Ampliamente reconocido por los usuarios guatemaltecos y con presencia en toda Centroamérica | Diseñado para servicios profesionales en Guatemala con afiliación inmediata |

Se elige NeoNet — Banco BI (Pay Bi) porque ofrece integración vía API con respaldo de Cybersource y Visa, lo que garantiza seguridad de nivel empresarial para los datos de pago de los pacientes. Al estar respaldado por Banco BI, uno de los bancos más sólidos de Guatemala, genera confianza tanto para la clínica como para los pacientes. Su proceso de afiliación es más formal que otras opciones, pero para un sistema médico que maneja información sensible, ese nivel de formalidad es una ventaja, no un obstáculo.

## Trade-offs

| Ventajas | Desventajas |
|---|---|
| Respaldado por Banco BI — uno de los bancos más sólidos de Guatemala | El proceso de afiliación es más formal y puede tomar más tiempo que otras opciones |
| Seguridad de nivel empresarial con respaldo de Cybersource y Visa | Las tarifas se negocian según el volumen — menos predecible que opciones con tarifa fija |
| Genera confianza en los pacientes por estar respaldado por un banco reconocido | Si hay problemas técnicos, la resolución puede ser más lenta que con fintechs |
| Integración vía API para sistemas digitales formales | |

## Integración

Como se muestra en el diagrama de despliegue, NeoNet es un servicio externo al que accede la aplicación principal para iniciar los cobros. Cuando el pago es procesado, NeoNet notifica al sistema a través de su API confirmando o rechazando la transacción. El módulo de Pagos y Facturación recibe esa respuesta, la traduce a un evento de dominio propio y lo publica en el bus de eventos interno para que los demás módulos reaccionen — Citas confirma la cita, Horarios bloquea el espacio, Notificaciones envía el correo.