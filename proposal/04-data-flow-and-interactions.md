# 04 — Flujos de Datos e Interacciones

Se describen los **3 flujos principales** del sistema. Cada uno incluye el escenario de negocio, un diagrama de secuencia, el camino feliz y al menos un camino de fallo o compensación.

---

## Flujo 1 — Agendamiento de Cita y Pago

### Escenario

Un paciente busca disponibilidad de un médico, selecciona un horario, completa el pago y recibe la confirmación de su cita. Este es el flujo central del sistema.

### Camino Feliz

```mermaid
sequenceDiagram
    actor Paciente
    participant API as API del Sistema
    participant HD as Horarios y Disponibilidad
    participant CR as Citas y Referidos
    participant PF as Pagos y Facturación
    participant NO as Notificaciones

    Paciente->>API: Busca disponibilidad del médico
    API->>HD: Consulta espacios disponibles
    HD-->>API: Lista de espacios disponibles
    API-->>Paciente: Muestra horarios disponibles

    Paciente->>API: Selecciona un horario
    API->>HD: Reserva el espacio temporalmente
    HD-->>API: Espacio reservado por 10 minutos
    API-->>Paciente: Confirma la reserva temporal

    Paciente->>API: Confirma la cita y envía datos de pago
    API->>CR: Crea la cita en estado PENDIENTE
    CR-->>API: Cita creada
    API->>PF: Inicia el cobro por el valor de la consulta
    PF-->>API: Cobro procesado exitosamente

    Note over PF,NO: A partir de aquí el flujo es asíncrono

    PF--)CR: Evento: PagoConfirmado
    CR--)HD: Evento: PagoConfirmado
    CR--)NO: Evento: CitaConfirmada

    CR-->>CR: Cambia cita a estado CONFIRMADA
    HD-->>HD: Cambia espacio a estado OCUPADO
    NO-->>Paciente: Envía correo de confirmación
    NO-->>Paciente: Programa recordatorio 24h antes
```

### Camino de Fallo — Pago Rechazado

```mermaid
sequenceDiagram
    actor Paciente
    participant API as API del Sistema
    participant PF as Pagos y Facturación
    participant CR as Citas y Referidos
    participant HD as Horarios y Disponibilidad
    participant NO as Notificaciones

    Paciente->>API: Confirma la cita y envía datos de pago
    API->>CR: Crea la cita en estado PENDIENTE
    API->>PF: Inicia el cobro

    PF-->>API: Cobro rechazado por fondos insuficientes
    API-->>Paciente: Informa que el pago no fue aprobado

    Note over PF,NO: El sistema compensa automáticamente

    PF--)CR: Evento: PagoFallido
    PF--)HD: Evento: PagoFallido
    PF--)NO: Evento: PagoFallido

    CR-->>CR: Cancela la cita automáticamente
    HD-->>HD: Libera el espacio reservado
    NO-->>Paciente: Envía correo informando el fallo y cómo reintentar
```

### Camino de Fallo — Espacio Expirado

```mermaid
sequenceDiagram
    actor Paciente
    participant HD as Horarios y Disponibilidad
    participant API as API del Sistema

    Note over HD: Han pasado 10 minutos sin confirmar

    HD-->>HD: Libera el espacio automáticamente
    HD--)HD: Evento: SlotLiberado

    Paciente->>API: Intenta confirmar la cita
    API->>HD: Verifica el espacio
    HD-->>API: El espacio ya no está disponible
    API-->>Paciente: Informa que el tiempo expiró y debe seleccionar otro horario
```

---

## Flujo 2 — Cancelación de Cita y Reembolso

### Escenario

Un paciente cancela una cita confirmada con más de 2 horas de anticipación. El sistema libera el espacio del médico, procesa el reembolso y notifica a ambas partes. Si el reembolso falla, el sistema lo registra para revisión manual.

### Camino Feliz

```mermaid
sequenceDiagram
    actor Paciente
    participant API as API del Sistema
    participant CR as Citas y Referidos
    participant PF as Pagos y Facturación
    participant HD as Horarios y Disponibilidad
    participant NO as Notificaciones

    Paciente->>API: Solicita cancelar la cita
    API->>CR: Verifica que la cancelación aplica sin penalización
    CR-->>CR: Confirma que faltan más de 2 horas para la cita
    CR-->>API: Cancelación aprobada
    API-->>Paciente: Confirma que la cita fue cancelada

    Note over CR,NO: A partir de aquí el flujo es asíncrono

    CR--)PF: Evento: CitaCancelada
    CR--)HD: Evento: CitaCancelada
    CR--)NO: Evento: CitaCancelada

    PF-->>PF: Procesa el reembolso al paciente
    HD-->>HD: Libera el espacio del médico
    NO-->>Paciente: Envía aviso de cancelación con información del reembolso
    NO-->>Paciente: Envía confirmación cuando el reembolso es procesado
```

### Camino de Fallo — Cancelación Fuera de Política

```mermaid
sequenceDiagram
    actor Paciente
    participant API as API del Sistema
    participant CR as Citas y Referidos

    Paciente->>API: Solicita cancelar la cita
    API->>CR: Verifica si aplica cancelación sin penalización
    CR-->>CR: Detecta que quedan menos de 2 horas para la cita
    CR-->>API: Cancelación fuera de política — no aplica reembolso
    API-->>Paciente: Informa que la cancelación a menos de 2 horas no tiene reembolso

    Note over CR: La cita permanece CONFIRMADA
    Note over CR: El pago no se devuelve
```

### Camino de Fallo — Reembolso Rechazado por el Proveedor

```mermaid
sequenceDiagram
    participant PF as Pagos y Facturación
    participant NO as Notificaciones

    PF-->>PF: Intenta procesar el reembolso
    PF-->>PF: El proveedor de pagos rechaza el reembolso
    PF-->>PF: Registra el caso para revisión manual del administrador

    PF--)NO: Evento: ReembolsoFallido
    NO-->>NO: Notifica al administrador para que revise el caso manualmente
```

---

## Flujo 3 — Médico Cancela y Pacientes en Lista de Espera son Notificados

### Escenario

Un médico marca un día como no disponible por una emergencia. El sistema cancela todas las citas confirmadas de ese día, reembolsa a los pacientes afectados, libera los espacios y notifica automáticamente a los pacientes que estaban en lista de espera para ese médico.

### Camino Feliz

```mermaid
sequenceDiagram
    actor Medico
    participant API as API del Sistema
    participant HD as Horarios y Disponibilidad
    participant CR as Citas y Referidos
    participant PF as Pagos y Facturación
    participant NO as Notificaciones

    Medico->>API: Registra excepción de no disponibilidad
    API->>HD: Bloquea todos los espacios del día afectado
    HD-->>HD: Marca espacios como BLOQUEADOS
    HD-->>API: Espacios bloqueados

    Note over HD,NO: El sistema reacciona en cadena de forma asíncrona

    HD--)CR: Evento: EspaciosBloqueados
    CR-->>CR: Cancela todas las citas confirmadas del día
    CR--)PF: Evento: CitasCanceladas (múltiples)
    CR--)NO: Evento: CitasCanceladas (múltiples)

    PF-->>PF: Procesa reembolsos para cada paciente afectado
    NO-->>NO: Envía aviso de cancelación a cada paciente afectado
    NO-->>NO: Envía aviso al médico confirmando la cancelación

    HD-->>HD: Revisa lista de espera para ese médico
    HD--)NO: Evento: PacientesEnEspera (lista de pacientes)
    NO-->>NO: Notifica a cada paciente en espera que hay espacios disponibles
```

### Camino de Fallo — Fallo al Notificar a Pacientes en Lista de Espera

```mermaid
sequenceDiagram
    participant NO as Notificaciones
    actor Paciente

    NO->>Paciente: Intento 1 de notificación — falla
    NO-->>NO: Reintento en 2 minutos

    NO->>Paciente: Intento 2 de notificación — falla
    NO-->>NO: Reintento en 8 minutos

    NO->>Paciente: Intento 3 de notificación — exitoso
    Paciente-->>Paciente: Recibe aviso de espacio disponible

    Note over NO: Si los 3 intentos fallan, se registra como NotificacionFallida
    Note over NO: El espacio sigue disponible para otros pacientes
```
