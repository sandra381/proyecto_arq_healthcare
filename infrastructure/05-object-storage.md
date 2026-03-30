# 05 — Almacenamiento de Archivos (Cloudinary)

## Responsabilidades

- Almacena los archivos del expediente clínico de cada paciente: recetas digitales, resultados de laboratorio e imágenes médicas
- Garantiza que los archivos estén cifrados y accesibles únicamente por el médico tratante y el propio paciente
- Genera enlaces temporales de acceso para que los archivos puedan descargarse de forma segura sin exponerlos públicamente
- Gestiona diferentes formatos de archivo: PDF para recetas, JPG y PNG para imágenes médicas

## Por Qué Este Proyecto lo Necesita

La base de datos PostgreSQL no está diseñada para almacenar archivos pesados — guardar imágenes o PDFs directamente en la base de datos la haría lenta y difícil de mantener. El sistema necesita un lugar separado y seguro donde guardar los documentos médicos de los pacientes.

En una clínica, el médico genera recetas digitales y los pacientes suben resultados de laboratorio que pueden pesar varios megabytes cada uno. Sin un servicio de almacenamiento dedicado, estos archivos saturarían la base de datos y afectarían el rendimiento de todo el sistema.

## Elección de Tecnología

| Dimensión | Cloudinary | AWS S3 | Almacenamiento en el servidor |
|---|---|---|---|
| **Administrado o instalado** | Administrado por Cloudinary | Administrado por AWS | Instalado y mantenido por el equipo |
| **Complejidad operativa** | Muy baja — interfaz simple y bien documentada | Baja — requiere configuración de permisos y políticas de acceso en AWS | Alta — el equipo es responsable de la seguridad, respaldos y espacio |
| **Costo para clínica pequeña-mediana** | Plan gratuito con 25GB de almacenamiento — suficiente para empezar, con planes de pago accesibles si crece | Se paga por almacenamiento y transferencia usados — sin costo fijo, escala bien a largo plazo | Sin costo de servicio, pero escalar implica migrar todos los archivos manualmente a un servidor más grande |
| **Ventaja principal** | Fácil de integrar desde el inicio y simple de escalar cambiando de plan sin modificar el código | Estándar de la industria con capacidad ilimitada y herramientas avanzadas para grandes volúmenes | Sin dependencia de servicios externos |

Se elige **Cloudinary** porque su plan gratuito cubre el volumen inicial de la clínica y su integración es más simple que AWS S3 para un equipo de 6 a 10 ingenieros. Si en el futuro la clínica crece significativamente, migrar a AWS S3 es posible sin cambiar el código del sistema — solo se cambia la configuración de dónde se guardan los archivos. Guardar archivos directamente en el servidor queda descartado porque no ofrece cifrado automático ni respaldos, lo cual es inaceptable para información médica sensible.

## Trade-offs

| Ventajas | Desventajas |
|---|---|
| Plan gratuito adecuado para el volumen de la clínica | Introduce dependencia con un servicio externo |
| Fácil de integrar sin configuración compleja | Si Cloudinary tiene una interrupción, los archivos no son accesibles |
| Cifrado automático de los archivos almacenados | Menos control sobre dónde se almacenan físicamente los datos |
| No requiere administración por parte del equipo | |

## Integración

Como se muestra en el diagrama de despliegue, Cloudinary es un servicio externo al que accede la aplicación principal cuando un médico genera una receta o un paciente sube un resultado de laboratorio. El sistema guarda en PostgreSQL únicamente la referencia al archivo — no el archivo en sí — y cuando el usuario necesita verlo, la aplicación solicita a Cloudinary un enlace temporal de acceso. Esto mantiene la base de datos liviana y los archivos seguros.
