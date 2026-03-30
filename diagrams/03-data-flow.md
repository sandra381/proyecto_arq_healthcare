# 03 — Flujos de Datos (Diagramas de Secuencia)

Se presentan 5 diagramas de secuencia que cubren los flujos más importantes del sistema. Cada diagrama muestra paso a paso cómo se comunican los módulos, incluyendo los caminos felices y los caminos de fallo.

---

## Diagrama 1 — Agendamiento de Cita (Parte 1: Búsqueda y Reserva)

### Descripción

Un paciente busca médicos disponibles por especialidad, selecciona un horario y reserva el espacio temporalmente mientras completa el pago. Esta parte del flujo es completamente síncrona — el paciente espera la respuesta en cada paso.

```mermaid
sequenceDiagram
    actor Paciente
    participant API as API del Sistema
    participant ME as Médicos y Especialidades
    participant HD as Horarios y Disponibilidad

    Paciente->>API: GET /medicos?especialidad=cardiologia
    API->>ME: Consulta médicos activos por especialidad
    ME-->>API: 200 OK {medicos: [{id, nombre, tarifa}]}
    API-->>Paciente: 200 OK {medicos: [{id, nombre, tarifa}]}

    Paciente->>API: GET /disponibilidad/{medicoId}?fecha=2025-06-10
    API->>HD: Consulta slots disponibles del médico
    HD-->>API: 200 OK {slots: [{id, hora, duracion}]}
    API-->>Paciente: 200 OK {slots: [{id, hora, duracion}]}

    Paciente->>API: POST /disponibilidad/reservar {slotId, pacienteId}
    API->>HD: Reserva el espacio temporalmente
    HD-->>API: 200 OK {slotId, expiraEn: 10min}
    API-->>Paciente: 200 OK {slotId, expiraEn: 10min}
```

---

## Diagrama 2 — Agendamiento de Cita (Parte 2: Pago y Confirmación)

### Descripción

Con el espacio reservado, el paciente confirma la cita y completa el pago. A partir de la confirmación del pago, el flujo se vuelve asíncrono — los módulos reaccionan al evento de pago confirmado de forma independiente.

```mermaid
sequenceDiagram
    actor Paciente
    participant API as API del Sistema
    participant CR as Citas y Referidos
    participant PF as Pagos y Facturación
    participant HD as Horarios y Disponibilidad
    participant NO as Notificaciones

    Paciente->>API: POST /citas {slotId, medicoId, motivoConsulta}
    API->>CR: Crea cita en estado PENDIENTE
    CR-->>API: 201 Created {citaId, estado: PENDIENTE}
    API-->>Paciente: 201 Created {citaId}

    Paciente->>API: POST /pagos/iniciar {citaId, metodoPagoId}
    API->>PF: Inicia el cobro por la consulta
    PF-->>API: 200 OK {pagoId, estado: PROCESANDO}
    API-->>Paciente: 200 OK {pagoId, estado: PROCESANDO}

    Note over PF,NO: Flujo asíncrono — los módulos reaccionan al evento

    PF-)CR: Evento: PagoConfirmado {citaId, monto}
    PF-)HD: Evento: PagoConfirmado {slotId}
    PF-)NO: Evento: PagoConfirmado {citaId, pacienteId}

    CR-->>CR: Cita cambia a estado CONFIRMADA
    HD-->>HD: Slot cambia a estado OCUPADO
    NO-->>Paciente: Correo: Confirmación de cita {fecha, medico, lugar}
    NO-->>Paciente: SMS: Recordatorio programado 24h antes
```

---

## Diagrama 3 — Agendamiento Fallido (Pago Rechazado)

### Descripción

El paciente intenta pagar pero el cobro es rechazado. El sistema cancela la cita automáticamente, libera el espacio del médico y notifica al paciente para que reintente con otro método de pago.

```mermaid
sequenceDiagram
    actor Paciente
    participant API as API del Sistema
    participant PF as Pagos y Facturación
    participant CR as Citas y Referidos
    participant HD as Horarios y Disponibilidad
    participant NO as Notificaciones

    Paciente->>API: POST /pagos/iniciar {citaId, metodoPagoId}
    API->>PF: Inicia el cobro por la consulta
    PF-->>API: 402 Payment Required {error: FONDOS_INSUFICIENTES}
    API-->>Paciente: 402 {mensaje: "El pago fue rechazado. Verifica tu método de pago."}

    Note over PF,NO: El sistema compensa automáticamente

    PF-)CR: Evento: PagoFallido {citaId, razon: FONDOS_INSUFICIENTES}
    PF-)HD: Evento: PagoFallido {slotId}
    PF-)NO: Evento: PagoFallido {citaId, pacienteId}

    CR-->>CR: Cita cambia a estado CANCELADA
    HD-->>HD: Slot vuelve a estado LIBRE
    NO-->>Paciente: Correo: Pago fallido {instrucciones para reintentar}
```

---

## Diagrama 4 — Cancelación de Cita y Reembolso

### Descripción

Un paciente cancela su cita con más de 2 horas de anticipación. El sistema verifica que aplica la política de cancelación sin penalización, libera el espacio del médico, procesa el reembolso y notifica a ambas partes.

```mermaid
sequenceDiagram
    actor Paciente
    participant API as API del Sistema
    participant CR as Citas y Referidos
    participant PF as Pagos y Facturación
    participant HD as Horarios y Disponibilidad
    participant NO as Notificaciones

    Paciente->>API: PUT /citas/{citaId}/cancelar {motivo}
    API->>CR: Solicita cancelación de la cita
    CR->>CR: Verifica anticipación mayor a 2 horas
    CR-->>API: 200 OK {estado: CANCELADA, reembolso: true}
    API-->>Paciente: 200 OK {mensaje: "Cita cancelada. El reembolso se procesará en breve."}

    Note over CR,NO: Flujo asíncrono — los módulos reaccionan al evento

    CR-)PF: Evento: CitaCancelada {citaId, monto}
    CR-)HD: Evento: CitaCancelada {slotId}
    CR-)NO: Evento: CitaCancelada {citaId, pacienteId, medicoId}

    PF-->>PF: Procesa reembolso al paciente
    HD-->>HD: Slot vuelve a estado LIBRE
    NO-->>Paciente: Correo: Cita cancelada y reembolso en proceso
    NO-->>Paciente: Correo: Reembolso procesado exitosamente
```

---

## Diagrama 5 — Médico Cancela y Notifica Lista de Espera

### Descripción

Un médico registra una excepción de no disponibilidad por emergencia. El sistema bloquea todos sus espacios del día, cancela las citas confirmadas, procesa los reembolsos correspondientes y notifica a los pacientes en lista de espera que hay espacios disponibles.

```mermaid
sequenceDiagram
    actor Medico
    participant API as API del Sistema
    participant HD as Horarios y Disponibilidad
    participant CR as Citas y Referidos
    participant PF as Pagos y Facturación
    participant NO as Notificaciones

    Medico->>API: POST /horarios/excepcion {medicoId, fecha, tipo: EMERGENCIA}
    API->>HD: Registra excepción y bloquea todos los espacios del día
    HD-->>API: 200 OK {espaciosBloqueados: 8}
    API-->>Medico: 200 OK {mensaje: "Día bloqueado. Las citas serán canceladas automáticamente."}

    Note over HD,NO: Flujo asíncrono en cadena

    HD-)CR: Evento: EspaciosBloqueados {medicoId, fecha, citasAfectadas}
    CR-->>CR: Cancela todas las citas confirmadas del día
    CR-)PF: Evento: CitasCanceladas {listaCitasIds, montos}
    CR-)NO: Evento: CitasCanceladas {pacientesAfectados}

    PF-->>PF: Procesa reembolsos para cada paciente afectado
    NO-->>NO: Envía aviso de cancelación a cada paciente afectado
    NO-->>Medico: Correo: Confirmación de bloqueo del día

    HD-->>HD: Revisa lista de espera para ese médico
    HD-)NO: Evento: SlotsDisponibles {pacientesEnEspera}
    NO-->>NO: Notifica a cada paciente en lista de espera
```
