# 04 — Diagrama de Despliegue e Infraestructura

## Descripción

Este diagrama muestra cómo está organizado físicamente el Healthcare Scheduling System cuando está corriendo en producción. Se divide en tres zonas: la red pública donde los usuarios acceden al sistema, la red interna donde corren los contenedores y la base de datos, y los servicios externos que el sistema usa para funcionar.

El sistema corre como un monolito modular en un contenedor principal, acompañado de workers en contenedores separados que ejecutan las tareas automáticas en segundo plano. Todos comparten una sola base de datos PostgreSQL y se comunican entre sí a través de RabbitMQ.

## Diagrama

```mermaid
graph TD
    subgraph INTERNET["Red Pública — Acceso de usuarios"]
        USR["Usuarios\nPacientes, Médicos, Recepcionistas, Admins"]
    end

    subgraph INTERNA["Red Interna"]
        LB["Balanceador de carga"]
        APP["Aplicación principal\nMonolito Modular"]
        W1["Worker de notificaciones"]
        W2["Worker de reportes y recordatorios"]
        W3["Worker de espacios expirados"]
        DB[("PostgreSQL\nBase de datos")]
        MQ[("RabbitMQ\nCola de mensajes")]
        LOG["Registro de logs y métricas"]
    end

    subgraph EXTERNOS["Servicios Externos"]
        AUTH["Auth0"]
        STR["NeoNet"]
        NOT["Servicio de notificaciones"]
        ALM["Servicio de almacenamiento"]
    end

    USR -->|HTTPS| LB
    LB -->|HTTP| APP
    APP -->|Lee y escribe| DB
    APP -->|Publica eventos| MQ
    APP -->|Registra actividad| LOG
    MQ -->|Consume eventos| W1
    MQ -->|Consume eventos| W2
    MQ -->|Consume eventos| W3
    W1 -->|Lee y escribe| DB
    W2 -->|Lee y escribe| DB
    W3 -->|Lee y escribe| DB
    APP -->|Verifica identidad| AUTH
    APP -->|Procesa cobros| STR
    W1 -->|Envía mensajes| NOT
    APP -->|Guarda archivos| ALM

    style USR fill:#5a3a8a,stroke:#3d2060,color:#ffffff
    style APP fill:#1a6b8a,stroke:#0d4f6b,color:#ffffff
    style LB fill:#1a6b8a,stroke:#0d4f6b,color:#ffffff
    style W1 fill:#1a6b8a,stroke:#0d4f6b,color:#ffffff
    style W2 fill:#1a6b8a,stroke:#0d4f6b,color:#ffffff
    style W3 fill:#1a6b8a,stroke:#0d4f6b,color:#ffffff
    style DB fill:#8a1a1a,stroke:#6b0d0d,color:#ffffff
    style MQ fill:#8a1a1a,stroke:#6b0d0d,color:#ffffff
    style LOG fill:#2d8a5e,stroke:#1a6b45,color:#ffffff
    style AUTH fill:#8a6a1a,stroke:#6b4f10,color:#ffffff
    style STR fill:#8a6a1a,stroke:#6b4f10,color:#ffffff
    style NOT fill:#8a6a1a,stroke:#6b4f10,color:#ffffff
    style ALM fill:#8a6a1a,stroke:#6b4f10,color:#ffffff
```

## Decisiones de Topología

**Monolito Modular con Event Driven**
Dentro de la aplicación principal, los módulos no se llaman directamente entre sí — se comunican a través de un bus de eventos interno. Por ejemplo, cuando el módulo de Pagos confirma un cobro, publica un evento interno que el módulo de Citas escucha para confirmar la cita. RabbitMQ se usa únicamente para la comunicación con los workers externos que corren en procesos separados.

**Balanceador de carga**
Recibe todo el tráfico de los usuarios y lo distribuye entre las instancias de la aplicación. Si el sistema necesita atender más usuarios, se agregan más instancias sin cambiar nada más.

**Aplicación principal y workers separados**
La aplicación principal atiende las solicitudes de los usuarios. Los workers se encargan de las tareas automáticas en segundo plano como notificaciones, reportes y liberar espacios expirados. Al estar separados, si un worker falla el sistema principal sigue funcionando normalmente.

**RabbitMQ como canal de mensajes**
La aplicación publica eventos y los workers los consumen cuando están disponibles. Si un worker está caído, RabbitMQ guarda los mensajes hasta que vuelva a funcionar — ningún mensaje se pierde.

**PostgreSQL como base de datos única**
Todos los componentes comparten la misma base de datos, pero cada módulo opera únicamente en su propio espacio. Es suficiente para el volumen de una clínica pequeña y mediana y mantiene el costo de infraestructura bajo.

**Sin caché**
El volumen de usuarios de clínicas pequeñas y medianas no justifica una capa de caché adicional en esta etapa. PostgreSQL responde con suficiente velocidad para este nivel de tráfico. Si el volumen crece en el futuro, se puede agregar sin cambiar la arquitectura.

**Observabilidad**
El sistema registra logs de toda la actividad y métricas de rendimiento. Esto permite identificar errores rápidamente y detectar cuando el sistema está bajo presión antes de que afecte a los usuarios.

