# Contextos Delimitados (DDD)

**MediSchedule** se descompone en 8 contextos delimitados. Cada uno representa un área de negocio independiente con sus propias entidades y eventos.

---

## Clasificación de Dominio

| Tipo | Contextos |
|---|---|
| **Dominio Central** | Agendamiento, Disponibilidad, Pagos |
| **Dominio de Soporte** | Expediente Médico, Notificaciones, Reportes y Analítica |
| **Dominio Genérico** | Identity & Access, Auditoría y Cumplimiento |

---

## 1. Identity & Access

**Propósito:** Registra usuarios, gestiona autenticación y controla el acceso según el rol de cada actor (Paciente, Médico, Recepcionista, Administrador).

**Entidades principales:**

| Entidad | Campos clave |
|---|---|
| `Usuario` | `id`, `email`, `passwordHash`, `estado: ACTIVO · SUSPENDIDO · PENDIENTE` |
| `Rol` | `id`, `nombre: PACIENTE · MEDICO · RECEPCIONISTA · ADMIN`, `permisos[]` |
| `Sesion` | `id`, `usuarioId`, `token`, `expiraEn` |

**Eventos que publica:** `UsuarioRegistrado` · `UsuarioVerificado` · `UsuarioSuspendido` · `SesionRevocada`

---

## 2. Agendamiento

**Propósito:** Gestiona el ciclo de vida de las citas médicas — reserva, confirmación, cancelación, reagendamiento e inasistencias.

**Entidades principales:**

| Entidad | Campos clave |
|---|---|
| `Cita` | `id`, `pacienteId`, `medicoId`, `slotId`, `estado: PENDIENTE · CONFIRMADA · CANCELADA · COMPLETADA · INASISTENCIA`, `tipo: PRESENCIAL · TELECONSULTA` |
| `RegistroCancelacion` | `id`, `citaId`, `motivo`, `canceladoPor: PACIENTE · MEDICO · SISTEMA` |
| `SolicitudReagendamiento` | `id`, `citaId`, `nuevoSlotId`, `estado: PENDIENTE · ACEPTADA · RECHAZADA` |

**Eventos que publica:** `CitaReservada` · `CitaConfirmada` · `CitaCancelada` · `CitaReagendada` · `CitaCompletada` · `InasistenciaRegistrada`

**Eventos que consume:** `SlotReservado` · `PagoConfirmado` · `PagoFallido` · `UsuarioVerificado`

---

## 3. Disponibilidad

**Propósito:** Administra los horarios de los médicos y la reserva de slots individuales para evitar dobles agendamientos.

**Entidades principales:**

| Entidad | Campos clave |
|---|---|
| `Horario` | `id`, `medicoId`, `plantillaSemanal`, `vigenciaDesde`, `vigenciaHasta` |
| `Slot` | `id`, `horarioId`, `inicio`, `fin`, `estado: DISPONIBLE · RESERVADO · OCUPADO · BLOQUEADO`, `duracionMinutos` |
| `PeriodoBloqueado` | `id`, `medicoId`, `motivo: VACACIONES · CONGRESO · CIERRE_CLINICA`, `inicio`, `fin` |

**Eventos que publica:** `SlotReservado` · `SlotLiberado` · `SlotOcupado` · `HorarioActualizado`

**Eventos que consume:** `CitaCancelada` · `CitaReagendada`

---

## 4. Pagos

**Propósito:** Gestiona el cobro de citas, reintentos automáticos ante fallos y reembolsos por cancelaciones.

**Entidades principales:**

| Entidad | Campos clave |
|---|---|
| `OrdenDePago` | `id`, `citaId`, `pacienteId`, `monto`, `moneda`, `estado: PENDIENTE · PROCESANDO · CONFIRMADO · FALLIDO · REEMBOLSADO` |
| `IntentoDePago` | `id`, `ordenId`, `referenciaStripe`, `resultado: EXITOSO · RECHAZADO · TIMEOUT`, `codigoError` |
| `Reembolso` | `id`, `ordenId`, `monto`, `motivo`, `estado: PENDIENTE · EMITIDO · RECHAZADO` |

**Eventos que publica:** `PagoConfirmado` · `PagoFallido` · `ReembolsoEmitido` · `FacturaGenerada`

**Eventos que consume:** `CitaReservada` · `CitaCancelada`

---

## 5. Notificaciones

**Propósito:** Envía comunicaciones transaccionales a pacientes y médicos por correo y SMS — confirmaciones, recordatorios y alertas de cancelación.

**Entidades principales:**

| Entidad | Campos clave |
|---|---|
| `SolicitudNotificacion` | `id`, `destinatarioId`, `canal: EMAIL · SMS`, `templateId`, `estado: EN_COLA · ENVIADA · FALLIDA` |
| `PlantillaNotificacion` | `id`, `clave`, `asunto`, `cuerpoTemplate`, `canal`, `locale` |
| `ReciboDeEntrega` | `id`, `solicitudId`, `idMensajeProveedor`, `entregadaEn`, `motivoFallo` |

**Eventos que publica:** `NotificacionEnviada` · `NotificacionFallida` · `RecordatorioProgramado`

**Eventos que consume:** `CitaReservada` · `CitaConfirmada` · `CitaCancelada` · `PagoConfirmado` · `PagoFallido` · `UsuarioRegistrado`

---

## 6. Expediente Médico

**Propósito:** Mantiene el historial clínico de cada paciente: notas de consulta, diagnósticos, prescripciones y archivos adjuntos.

**Entidades principales:**

| Entidad | Campos clave |
|---|---|
| `ExpedientePaciente` | `id`, `pacienteId`, `grupoSanguineo`, `alergias[]`, `creadoEn` |
| `NotaDeConsulta` | `id`, `citaId`, `medicoId`, `resumen`, `diagnostico`, `prescripciones`, `bloqueada: Boolean` |
| `ArchivoMedico` | `id`, `notaId`, `claveS3`, `tipoMime`, `tamanoBytes` |

**Eventos que publica:** `NotaDeConsultaCreada` · `NotaDeConsultaBloqueada` · `ArchivoMedicoSubido`

**Eventos que consume:** `CitaCompletada` · `UsuarioVerificado`

---

## 7. Reportes y Analítica

**Propósito:** Provee a administradores métricas operativas agregadas: ocupación de agenda, inasistencias e ingresos por período.

**Entidades principales:**

| Entidad | Campos clave |
|---|---|
| `SnapshotOcupacionDiaria` | `id`, `medicoId`, `fecha`, `totalSlots`, `slotsReservados`, `slotsCompletados`, `slotsInasistencia` |
| `ReporteDeIngresos` | `id`, `periodoInicio`, `periodoFin`, `ingresoTotal`, `montoReembolsado` |
| `SuscripcionReporte` | `id`, `suscriptorId`, `tipoReporte`, `canalEntrega: EMAIL · DASHBOARD`, `proximaEjecucion` |

**Eventos que publica:** `ReporteGenerado`

**Eventos que consume:** `CitaCompletada` · `InasistenciaRegistrada` · `PagoConfirmado` · `ReembolsoEmitido`

---

## 8. Auditoría y Cumplimiento

**Propósito:** Registra un log inmutable de todas las acciones del sistema para cumplimiento normativo y resolución de disputas sobre datos médicos.

**Entidades principales:**

| Entidad | Campos clave |
|---|---|
| `EventoDeAuditoria` | `id`, `ocurridoEn`, `actorId`, `rolActor`, `accion`, `tipoRecurso`, `recursoId`, `payload (PII redactada)` |
| `BanderaDeIncumplimiento` | `id`, `eventoId`, `tipo: ACCESO_NO_AUTORIZADO · PATRON_SOSPECHOSO · EXPORTACION_DATOS`, `resolucion` |
| `PoliticaDeRetencion` | `id`, `tipoRecurso`, `anosRetencion`, `ultimaAplicacionEn` |

**Eventos que publica:** `EventoDeAuditoriaRegistrado` · `BanderaDeIncumplimientoLanzada`

**Eventos que consume:** Todos los eventos significativos de los demás contextos

---

## Relaciones entre Contextos

| Contexto | Depende de | Patrón |
|---|---|---|
| Agendamiento | Identity, Disponibilidad, Pagos | ACL |
| Disponibilidad | Identity | OHS/PL |
| Pagos | Identity, Agendamiento | ACL hacia Stripe |
| Notificaciones | Identity, Agendamiento, Pagos | CF hacia SendGrid/Twilio |
| Expediente Médico | Identity, Agendamiento | Partnership con Agendamiento |
| Reportes | Agendamiento, Pagos, Disponibilidad | ACL (modelo de lectura propio) |
| Auditoría | Todos los contextos | ACL (sink inmutable) |
| Identity | — | OHS/PL (raíz del sistema) |
