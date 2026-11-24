---
title: "Criterios de arquitectura"
study_order: 6
permalink: /fundamentos/06-criterios-de-arquitectura.html
---

# 06. Cómo razonar sobre una arquitectura

**Al terminar podrás** convertir un requisito en una decisión técnica, diferenciar disponibilidad y recuperación, y explicar qué sacrificas al escoger una opción.

## Traducir una petición a restricciones

Cuando alguien pide que una aplicación sea «rápida, segura y barata», averigua qué significan esas palabras en esa aplicación. «Debe seguir aceptando pedidos durante el fallo de una AZ» es un requisito de continuidad concreto. «Un informe puede tardar cinco minutos, pero el cliente necesita confirmar que se registró en dos segundos» separa la latencia de entrega del trabajo del tiempo de procesado. «Los datos personales deben permanecer en una región aprobada» limita los destinos de almacenamiento y de recuperación.

Un método sencillo para resolver preguntas de examen y decisiones reales:

1. Lee la última pregunta y detecta qué optimiza, por ejemplo *most cost-effective*, *least operational overhead* o *lowest latency*.
2. Marca requisitos que eliminan opciones. Si se exige *automatic failover*, una promoción manual puede quedar fuera incluso si es barata. Si se deben compartir archivos POSIX entre varias máquinas, la interfaz de archivos importa.
3. Traza el flujo mínimo que recorre entrada, ejecución, estado, datos y dependencias. Sigue la identidad que llama a cada API.
4. Comprueba qué pasa durante un fallo y al crecer la carga. Indica qué componente debe seguir funcionando y qué estado necesita.
5. Compara las opciones que pasan los filtros. Cuando varias cumplen, decide según coste, complejidad operativa, rendimiento u otro criterio explícito.

En un escenario real puedes pedir datos de carga, presupuesto y tiempos. En una pregunta de examen trabajas con lo que te da el enunciado. Las palabras como *least*, *only*, *must*, *at scale* y *without* contienen restricciones que cambian la respuesta.

## Conceptos que se confunden

| Concepto | Qué mide o describe | Ejemplo |
| --- | --- | --- |
| **Availability** | Qué parte del tiempo o de las peticiones puede atender el servicio en las condiciones acordadas. | La tienda debería poder aceptar pedidos durante un fallo de instancia. |
| **Durability** | Qué bien se conserva lo que se almacenó. | Un objeto puede seguir intacto aunque se produzca un fallo temporal de acceso. |
| **Resilience** | Capacidad de resistir fallos y recuperar la prestación del servicio. | Puedes aislar el problema, redirigir tráfico, recuperar datos y volver a la normalidad. |
| **Fault tolerance** | Capacidad de continuar operando si falla un componente contemplado. | La pérdida de una instancia deja otra capaz de atender. El impacto medido depende del diseño. |
| **Scalability** | Capacidad de aumentar el trabajo que puede gestionar el sistema. | Añadir instancias a una capa web. |
| **Elasticity** | Ajustar la capacidad al subir y bajar la demanda. | Añadir workers con cola acumulada y reducirlos cuando se vacía. |
| **Latency** | Lo que tarda una operación desde el punto de vista que estás midiendo. | Tiempo desde que llega una petición hasta que recibe su respuesta. |
| **Throughput** | Cantidad de trabajo que se termina en un periodo. | Informes generados por minuto. |

**Escalado vertical** es dar más capacidad a una unidad, por ejemplo más RAM a una máquina. **Escalado horizontal** es añadir unidades. Antes de escalar, identifica el cuello de botella. Si la base ya está saturada, añadir más servidores de aplicación puede aumentar su presión. Si la aplicación mantiene sesiones locales, repartir peticiones entre máquinas requiere resolver cómo se conserva ese estado. Se puede mover a un servicio apropiado o rediseñar el flujo según los requisitos.

**Latency y throughput pueden moverse de forma distinta.** Una cola puede sostener una entrada alta y responder rápido con un acuse. Si el backlog crece durante mucho tiempo, los trabajos esperan más para finalizar. Es posible cumplir la latencia de aceptación y fallar la latencia de entrega final.

## Alta disponibilidad, backups y recuperación

**High availability (HA)** reduce la interrupción ante los fallos de componentes que cubre el diseño. Un ejemplo es servir desde dos AZ y mantener capacidad para que una sola absorba el tráfico. **Disaster recovery (DR)** define cómo recuperar la carga ante un desastre con un objetivo de tiempo y de datos. Puede usar otra región según el escenario.

- **Recovery Time Objective (RTO)** es el tiempo máximo de interrupción que el negocio está dispuesto a tolerar. Cuenta detección, recuperación de datos, puesta en marcha de dependencias, cambio de tráfico y verificación.
- **Recovery Point Objective (RPO)** es el máximo retroceso aceptable de los datos, medido en tiempo. Un RPO de 15 minutos significa que tras el incidente deberías poder recuperar un punto como máximo 15 minutos anterior al fallo.

Supón que un servicio falla a las 11:00. El último punto recuperable es de las 10:44 y se vuelve a atender al público a las 11:35. El retroceso es de 16 minutos y la interrupción de 35. Con objetivos de **RPO 15 min y RTO 60 min**, cumple el tiempo de recuperación y supera en un minuto la pérdida de datos que se había aceptado. Es posible cumplir uno y fallar el otro.

La réplica de una base ayuda a mantener disponibilidad si falla infraestructura. Un borrado lógico puede replicarse. Una copia recuperable o un punto en el tiempo cubren otro caso. En **RDS Multi-AZ DB instance**, la réplica en espera apoya el failover y no se usa para servir lecturas. Una **read replica** típica sirve para escalar lecturas y suele replicar de forma asíncrona. Un **Multi-AZ DB cluster** es una configuración diferente y puede incluir instancias lectoras. En una pregunta, fíjate en si dice *DB instance deployment*, *read replica* o *DB cluster*.

Para reconocer estrategias en una segunda región:

| Estrategia | Qué hay en la ubicación de recuperación | Coste y tiempo relativos |
| --- | --- | --- |
| **Backup and restore** | Copias recuperables y forma de recrear la infraestructura. | Menor coste de capacidad en espera y mayor tiempo de puesta en marcha. |
| **Pilot light** | Núcleo de la infraestructura, habitualmente replicación de datos, con capas que se levantan tras el fallo. | Reduce parte del tiempo de puesta en marcha a cambio de recursos activos. |
| **Warm standby** | Una versión funcional del sistema con capacidad reducida. | Puede atender con capacidad inicial y escalar, con mayor coste continuo. |
| **Multi-site active/active** | Varias ubicaciones ya sirven tráfico. | Puede minimizar la interrupción y exige resolver consistencia, capacidad, encaminamiento y failover. |

El RPO y el RTO efectivos dependen de la implementación y las pruebas. Una arquitectura dibujada con dos regiones todavía puede fallar si las claves, DNS, permisos, imágenes o despliegues necesarios dependen del mismo punto de fallo.

## Los seis pilares de AWS Well-Architected

| Pilar | Qué pregunta harías |
| --- | --- |
| **Operational Excellence** | ¿Cómo desplegamos, observamos, respondemos y aprendemos de incidencias? |
| **Security** | ¿Qué identidades interactúan con qué datos y cómo validamos sus controles? |
| **Reliability** | ¿Qué fallos asumimos y cómo seguimos operando o recuperamos? |
| **Performance Efficiency** | ¿Qué recurso se adapta a la latencia, al throughput y al patrón de demanda requeridos? |
| **Cost Optimization** | ¿Qué capacidad y servicios necesitamos, y por qué se nos factura? |
| **Sustainability** | ¿Cuánto recurso necesita la carga por unidad de trabajo y cómo reducimos desperdicio? |

Los pilares sirven para revisar un diseño desde varios ángulos. Los **cuatro dominios del SAA-C03** son la clasificación del examen. Te puedes apoyar en los pilares para justificar una respuesta aunque la pregunta pertenezca a uno de los cuatro dominios.

## Dos casos resueltos

**Catálogo público con mucho tráfico de lectura.** Los usuarios de varias ubicaciones descargan las mismas imágenes y deben poder ver el catálogo con baja latencia. Guardaría imágenes en S3 y serviría las peticiones con CloudFront. La caché cerca de los usuarios reduce viajes al origen. Para fichas privadas o personalizadas, revisaría claves de caché, autorización y qué se puede compartir entre usuarios. El hecho de que una CDN acelere recursos públicos no justifica cachear todos los datos de la misma forma.

**Informes trimestrales con demanda irregular.** El equipo necesita SQL y recibe de golpe cientos de peticiones para calcular informes. Mantendría el motor relacional si las consultas lo requieren. Si el resultado puede llegar más tarde, una API puede encolar los trabajos y un grupo de workers ejecutarlos aparte con un límite que proteja la base. Para decidir cuánta capacidad usar, mediría tiempo de espera en cola, consumo de la base y cuántos informes se terminan por minuto.

## Fuentes

Consultadas el 24 de noviembre de 2026.

- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html) y [conceptos del marco](https://docs.aws.amazon.com/wellarchitected/latest/framework/concepts.html)
- [Pilar de fiabilidad](https://docs.aws.amazon.com/wellarchitected/latest/reliability-pillar/) y [estrategias de recuperación](https://docs.aws.amazon.com/wellarchitected/2024-06-27/framework/rel_planning_for_recovery_disaster_recovery.html)
- [RDS Multi-AZ DB instance](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/Concepts.MultiAZSingleStandby.html), [réplicas de lectura](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_ReadRepl.html) y [conexiones en Multi-AZ DB cluster](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/multi-az-db-clusters-concepts-connection-management.html)
- [Cómo CloudFront entrega contenido](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/HowCloudFrontWorks.html)

[Índice de estudio](../README.md) · [Lectura anterior](05-mapa-de-servicios.md) · [Lectura siguiente](07-costes-y-entornos.md)
