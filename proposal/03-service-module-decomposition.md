# 03 — Descomposición de Módulos y Servicios

## Estructura del Repositorio

El sistema está organizado en un único repositorio con 9 módulos, cada uno correspondiente a un contexto delimitado. La regla principal es que cada módulo solo puede usar lo que otro módulo expone en su carpeta `publico/` — nunca puede acceder a su código interno.

```
healthcare-scheduling-system/
│
├── src/
│   ├── modulos/
│   │   ├── usuarios-acceso/
│   │   │   ├── publico/              # Lo que otros módulos pueden usar
│   │   │   ├── interno/
│   │   │   │   ├── entidades/        # Usuario, SesionActiva, PermisoRol
│   │   │   │   ├── repositorios/     # Guarda y consulta datos de usuarios
│   │   │   │   ├── servicios/        # Lógica de registro e inicio de sesión
│   │   │   │   └── manejadores/      # Reacciona a eventos de otros módulos
│   │   │   ├── api/
│   │   │   │   └── rutas.ts          # POST /registro, POST /login, PUT /perfil
│   │   │   └── modulo.ts             # Configuración e inicio del módulo
│   │   │
│   │   ├── medicos-especialidades/
│   │   │   ├── publico/
│   │   │   ├── interno/
│   │   │   │   ├── entidades/        # Medico, Especialidad, CalificacionMedico
│   │   │   │   ├── repositorios/     # Guarda y consulta datos de médicos
│   │   │   │   ├── servicios/        # Lógica de médicos y especialidades
│   │   │   │   └── manejadores/      # Crea perfil médico cuando se registra un médico
│   │   │   ├── api/
│   │   │   │   └── rutas.ts          # GET /medicos, GET /especialidades
│   │   │   └── modulo.ts
│   │   │
│   │   ├── horarios-disponibilidad/
│   │   │   ├── publico/
│   │   │   ├── interno/
│   │   │   │   ├── entidades/        # HorarioMedico, SlotDisponible, ListaEspera
│   │   │   │   ├── repositorios/     # Guarda y consulta horarios y espacios
│   │   │   │   ├── servicios/        # Lógica de espacios disponibles y lista de espera
│   │   │   │   ├── traductor/        # Convierte información de otros módulos al formato propio
│   │   │   │   └── tareas/           # Libera espacios que expiraron sin confirmarse
│   │   │   ├── api/
│   │   │   │   └── rutas.ts          # GET /disponibilidad/:medicoId, POST /reservar
│   │   │   └── modulo.ts
│   │   │
│   │   ├── citas-referidos/
│   │   │   ├── publico/
│   │   │   ├── interno/
│   │   │   │   ├── entidades/        # Cita, HistorialCita, Referido
│   │   │   │   ├── repositorios/     # Guarda y consulta citas y referidos
│   │   │   │   ├── servicios/        # Lógica de agendamiento y referidos
│   │   │   │   └── manejadores/      # Confirma la cita cuando el pago es aprobado
│   │   │   ├── api/
│   │   │   │   └── rutas.ts          # POST /citas, PUT /citas/:id/cancelar
│   │   │   └── modulo.ts
│   │   │
│   │   ├── pagos-facturacion/
│   │   │   ├── publico/
│   │   │   ├── interno/
│   │   │   │   ├── entidades/        # Pago, MetodoPago, Reembolso
│   │   │   │   ├── repositorios/     # Guarda y consulta pagos y reembolsos
│   │   │   │   ├── servicios/        # Lógica de cobros y devoluciones
│   │   │   │   ├── traductor/        # Convierte respuestas del proveedor de pagos a eventos propios
│   │   │   │   └── webhooks/         # Recibe confirmaciones de pago del proveedor externo
│   │   │   ├── api/
│   │   │   │   └── rutas.ts          # POST /pagos/iniciar, GET /pagos/:id
│   │   │   └── modulo.ts
│   │   │
│   │   ├── notificaciones/
│   │   │   ├── publico/
│   │   │   ├── interno/
│   │   │   │   ├── entidades/        # Plantilla, EnvioNotificacion, RegistroEntrega
│   │   │   │   ├── repositorios/     # Guarda y consulta envíos y registros
│   │   │   │   ├── servicios/        # Lógica de envío por correo, SMS y notificación push
│   │   │   │   └── tareas/           # Procesa la cola de mensajes pendientes y recordatorios
│   │   │   ├── api/
│   │   │   │   └── rutas.ts          # Rutas internas de administración
│   │   │   └── modulo.ts
│   │   │
│   │   ├── historial-clinico/
│   │   │   ├── publico/
│   │   │   ├── interno/
│   │   │   │   ├── entidades/        # Expediente, NotaConsulta, RecetaDigital, ResultadoLaboratorio
│   │   │   │   ├── repositorios/     # Guarda y consulta expedientes y documentos médicos
│   │   │   │   ├── servicios/        # Lógica de expedientes y recetas digitales
│   │   │   │   └── traductor/        # Convierte eventos de citas al formato clínico propio
│   │   │   ├── api/
│   │   │   │   └── rutas.ts          # GET /expediente/:pacienteId, POST /notas
│   │   │   └── modulo.ts
│   │   │
│   │   ├── reportes/
│   │   │   ├── publico/
│   │   │   ├── interno/
│   │   │   │   ├── entidades/        # ResumenDiario, ReporteMedico, ReporteIngreso
│   │   │   │   ├── repositorios/     # Guarda y consulta reportes generados
│   │   │   │   ├── servicios/        # Lógica de generación de reportes
│   │   │   │   └── tareas/           # Genera reportes diarios y mensuales de forma automática
│   │   │   ├── api/
│   │   │   │   └── rutas.ts          # GET /reportes/ingresos, GET /reportes/medicos
│   │   │   └── modulo.ts
│   │   │
│   │   └── inventario/
│   │       ├── publico/
│   │       ├── interno/
│   │       │   ├── entidades/        # Medicamento, MovimientoStock, AlertaStock
│   │       │   ├── repositorios/     # Guarda y consulta medicamentos y movimientos
│   │       │   ├── servicios/        # Lógica de control de stock y alertas
│   │       │   └── manejadores/      # Descuenta stock cuando se emite una receta
│   │       ├── api/
│   │       │   └── rutas.ts          # GET /medicamentos, POST /movimientos
│   │       └── modulo.ts
│   │
│   ├── compartido/                   # Código disponible para todos los módulos
│   │   ├── eventos/                  # Canal de comunicación entre módulos
│   │   ├── errores/                  # Errores comunes del sistema
│   │   ├── autenticacion/            # Verificación de token y permisos por rol
│   │   ├── base-datos/               # Conexión compartida a la base de datos
│   │   └── utilidades/               # Manejo de fechas, zonas horarias y paginación
│   │
│   └── app.ts                        # Punto de entrada: inicia todos los módulos
│
├── pruebas/
│   ├── unitarias/                    # Pruebas por módulo de forma aislada
│   ├── integracion/                  # Pruebas con base de datos real
│   └── e2e/                          # Pruebas del flujo completo de principio a fin
│
├── configuracion/
│   ├── local/                        # Configuración para correr el sistema en la computadora del desarrollador
│   └── produccion/                   # Configuración para el servidor en producción
│
└── README.md
```

---

## Propiedad y Exposición por Módulo

Cada módulo controla un área específica del negocio y solo comparte con los demás lo estrictamente necesario.

| Módulo | Qué controla | Qué comparte con otros módulos | Contexto |
|---|---|---|---|
| `usuarios-acceso` | Cuentas, sesiones y permisos por tipo de usuario | Datos básicos del usuario y verificación de identidad | Usuarios y Acceso |
| `medicos-especialidades` | Perfiles de médicos, especialidades y calificaciones | Información del médico y sus especialidades | Médicos y Especialidades |
| `horarios-disponibilidad` | Horarios semanales, espacios disponibles y lista de espera | Espacios de tiempo disponibles para agendar | Horarios y Disponibilidad |
| `citas-referidos` | Ciclo de vida de citas, historial de cambios y referidos | Información de la cita y su estado actual | Citas y Referidos |
| `pagos-facturacion` | Cobros, métodos de pago y reembolsos | Estado del pago de una cita | Pagos y Facturación |
| `notificaciones` | Plantillas, envíos y registros de mensajes | No comparte nada — solo recibe eventos de otros módulos | Notificaciones |
| `historial-clinico` | Expedientes, notas, recetas y resultados de laboratorio | Datos del expediente y notas de consulta | Historial Clínico |
| `reportes` | Resúmenes diarios y reportes de ingresos | Datos consolidados para administradores | Reportes |
| `inventario` | Medicamentos, entradas, salidas y alertas de stock | Datos de medicamentos disponibles | Inventario |

---

## Cómo se Evita que los Módulos se Mezclen

### 1. Solo se puede usar lo que está en la carpeta público

Si un módulo quiere usar algo de otro, solo puede importarlo desde su carpeta `publico/`. Intentar acceder a la carpeta `interno/` de otro módulo es detectado automáticamente y bloquea la integración del cambio.

```typescript
//Correcto — usar lo que el módulo expone públicamente
import { CitaDto } from '@modulos/citas-referidos/publico';

//Prohibido — acceder al código interno de otro módulo
import { CitaRepositorio } from '@modulos/citas-referidos/interno/repositorios';
```

### 2. Cada módulo tiene su propio espacio en la base de datos

Aunque todos los módulos comparten la misma base de datos, cada uno guarda y consulta información únicamente en su propio espacio. No existen conexiones directas entre los espacios de distintos módulos.

```
Espacio usuarios          → solo módulo usuarios-acceso
Espacio medicos           → solo módulo medicos-especialidades
Espacio horarios          → solo módulo horarios-disponibilidad
Espacio citas             → solo módulo citas-referidos
Espacio pagos             → solo módulo pagos-facturacion
Espacio notificaciones    → solo módulo notificaciones
Espacio historial_clinico → solo módulo historial-clinico
Espacio reportes          → solo módulo reportes
Espacio inventario        → solo módulo inventario
```

### 3. Los módulos se comunican solo por eventos

Cuando un módulo necesita reaccionar a algo que ocurrió en otro, lo hace escuchando un evento — nunca llamando directamente al código interno del otro módulo. Esto mantiene los módulos independientes entre sí.

```typescript
// El módulo de inventario escucha cuando se emite una receta
// y descuenta automáticamente los medicamentos del stock
@ManejaEvento(RecetaEmitida)
async manejar(evento: RecetaEmitida): Promise<void> {
  await this.inventarioServicio.descontarStock(evento.medicamentos);
}
```

---

## Qué corre en el mismo proceso y qué corre separado

Todos los módulos corren juntos en el proceso principal de la aplicación. Las tareas automáticas son la excepción — corren en procesos separados para no afectar el tiempo de respuesta de los usuarios.

| Componente | ¿Proceso separado? | Razón |
|---|---|---|
| Todos los módulos (API) | No — proceso principal | Las solicitudes son rápidas y no afectan el rendimiento general |
| Tarea: espacios expirados | Sí | Revisa y libera espacios vencidos cada minuto |
| Tarea: notificaciones | Sí | Llama a servicios externos con tiempos de respuesta variables |
| Tarea: recordatorios | Sí — cada hora | Programa los recordatorios del día siguiente |
| Tarea: reporte diario | Sí | En horario de baja actividad | Hace consultas pesadas sobre toda la base de datos |
| Tarea: reporte mensual | Sí — primer día de cada mes | Genera los reportes de ingresos para los administradores |
