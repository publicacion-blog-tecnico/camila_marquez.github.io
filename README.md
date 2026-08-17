# camila_marquez.github.io
# Entregable-Mentalidad-de-Crecimiento-y-Communicacion-en-Entornos-Digitales
Este repositorio contiene la entrega correspondiente a la materia Mentalidad de Crecimiento y Comunicación en Entornos Digitales, en el marco de la carrera de Marketing Digital.
gitcommit-m  Cómo resolvimos un fallo de rendimiento en producción mediante análisis de causa raíz
Contexto
Durante el último trimestre, nuestro equipo estuvo trabajando en el escalamiento de la plataforma de comercio electrónico de la empresa. El entorno de producción opera con una arquitectura de microservicios alojada en Kubernetes, procesando un término medio de 5,000 solicitudes por minuto. La base de datos principal es PostgreSQL.

Problema
Durante el pico de tráfico del pasado "Black Friday", el tiempo de respuesta del servicio de pagos aumentó drásticamente de 200 ms a más de 8,5 segundos, causando tiempos de espera agotados (timeouts) y carritos abandonados.

A nivel técnico, la utilización de CPU del pod de pagos se mantuvo estable al 30%, pero las conexiones a la base de datos PostgreSQL se saturaron al 100% de su capacidad en cuestión de minutos.

Acciones (Post-Mortem Constructivo)
Para abordar esta situación sin buscar culpables y enfocándonos en el aprendizaje del sistema, realizamos las siguientes acciones:

Mitigación inmediata: Escalamos temporalmente el connection pool de la base de datos y aplicamos un límite de tasa (rate limit) en la pasarela de pagos para estabilizar el sistema.

Análisis de causa raíz (Los 5 Porqués):

¿Por qué se saturaron las conexiones? Las consultas SQL tardaban demasiado en responder.

¿Por qué tardaban tanto? Se ejecutaba una consulta SELECT * sobre la tabla de transacciones sin utilizar un índice adecuado para filtrar por status y user_id.

¿Por qué no había un índice? La migración de base de datos que agregaba el índice no se incluyó en el último despliegue.

Medidas correctivas:

Creada e implementada la migración de base de datos para incluir el índice idx_transactions_user_status.

Refactorizado el código del ORM para seleccionar únicamente los campos necesarios en lugar de SELECT *.

Implementadas pruebas de carga automatizadas en el pipeline de CI/CD que simulan un volumen de tráfico 3x superior al pico esperado.

Aprendizajes
Observabilidad desde el día 1: No basta con monitorear la CPU/Memoria; las métricas a nivel de base de datos y la tasa de errores en las consultas son críticas.

Automatización de migraciones: Las migraciones de base de datos deben estar estrictamente ligadas y validadas dentro del ciclo de despliegue automatizado.

Cultura de Post-Mortem Blameless (Sin Culpas): Analizar el fallo como un problema del sistema y no de la persona que escribió el código permitió al equipo actuar rápido, colaborar abiertamente y prevenir que vuelva a ocurrir.
