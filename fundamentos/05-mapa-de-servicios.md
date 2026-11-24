---
title: "Mapa de servicios"
study_order: 5
permalink: /fundamentos/05-mapa-de-servicios.html
---

# 05. Mapa inicial de servicios AWS

**Al terminar podrás** reconocer qué familia resuelve cada problema y explicar por qué dos servicios de aspecto similar cumplen requisitos distintos. Vuelve a este mapa cuando aparezca un nombre nuevo en un escenario.

## Dónde se ejecuta el código

| Servicio | Qué es | Cuándo aparece |
| --- | --- | --- |
| **Amazon EC2** | Instancias virtuales con capacidad seleccionable. | Control del SO invitado, software que espera un servidor, servicios de larga duración y configuraciones específicas. |
| **Amazon ECS** | Orquestador de contenedores de AWS. | Desplegar tareas de contenedor, mantener un número de réplicas y gestionar su ciclo de vida. |
| **Amazon EKS** | Servicio gestionado de Kubernetes. | Una aplicación o equipo que requiere APIs, ecosistema y prácticas de Kubernetes. |
| **AWS Fargate** | Capacidad gestionada para ejecutar contenedores con ECS o EKS en modos compatibles. | Ejecutar contenedores sin administrar los hosts que los alojan. |
| **AWS Lambda** | Entorno que ejecuta código al invocar una función. | Trabajo acotado y activado por una petición o evento, como transformar un fichero al subirlo. |

ECS y Fargate encajan en el mismo diseño. ECS orquesta y Fargate proporciona capacidad. **Amazon ECR** almacena imágenes de contenedor para que puedas distribuirlas. Guardar una imagen en ECR y ejecutarla son operaciones distintas. Lambda también admite empaquetar código en una imagen, pero el modelo de ejecución sigue siendo el de una función.

**Pregunta decisiva.** ¿La aplicación necesita controlar el sistema operativo, mantener un proceso de larga duración o reaccionar a trabajo discreto? Un agente que instala módulos de kernel apunta a control del host. Una API Docker con un proceso persistente puede encajar en ECS. Una función que genera una miniatura cuando entra una imagen puede encajar en Lambda si sus límites se ajustan al trabajo.

## Dónde guardas datos

| Modelo | Ejemplo AWS | Interfaz y uso |
| --- | --- | --- |
| **Bloques** | **Amazon EBS** | Volumen que una instancia de la misma AZ usa como disco. Encima puedes crear un sistema de archivos o emplear bloques desde una aplicación compatible. |
| **Archivos** | **Amazon EFS** | Sistema de archivos NFS gestionado. Varios clientes compatibles pueden montar y usar el mismo sistema de archivos. |
| **Objetos** | **Amazon S3** | Cada objeto se ubica por bucket y clave. La aplicación usa una API para guardar, leer o enumerar objetos. Apropiado para copias, imágenes y contenido distribuido. |
| **Relacional** | **Amazon RDS**, incluida **Amazon Aurora** como opción de base relacional de AWS | Maneja SQL y relaciones. Encaja cuando el motor y las transacciones de una base relacional son requisitos. |
| **Clave–valor / documentos** | **Amazon DynamoDB** | Organiza el acceso a datos a partir de claves e índices. Se diseña a partir de los patrones de consulta, como buscar una sesión por ID. |

Un proceso Linux que hace `open()` sobre un archivo de un montaje NFS espera operaciones de sistema de archivos. Un cliente S3 llama a operaciones como `GetObject` sobre una clave. Si una aplicación necesita compartir un montaje entre varias instancias, EFS es una opción natural. Si necesita descargar un informe identificándolo por su clave, S3 encaja. Si las consultas necesitan unir pedidos, clientes y pagos existentes en PostgreSQL, RDS PostgreSQL conserva el modelo relacional y el motor. DynamoDB cobraría sentido si los patrones de acceso y la aplicación están diseñados para ese modelo.

**Primer vocabulario de S3.** El **bucket** contiene objetos. La **key** identifica el objeto dentro del bucket. Un **prefix** es el comienzo de la clave. En `informes/2026/ventas.csv`, `informes/2026/` es un prefijo útil para organizar o acotar operaciones. Si activas **versioning** y subes dos veces la misma clave, puedes recuperar versiones diferentes. Con versionado habilitado, un borrado normal añade un **delete marker**. La lectura corriente deja de mostrar el objeto y las versiones anteriores permanecen hasta que se eliminen o caduquen según la configuración. Recuperarlas requiere permisos adecuados.

## Cómo entra y circula el tráfico

- **Amazon VPC** crea un entorno lógico de red dentro de una región. Las subredes pertenecen a una AZ. Sus tablas de rutas indican dónde enviar el tráfico.
- **Internet gateway** conecta una VPC con internet. En el caso típico IPv4, una instancia que se comunica directamente necesita además una IP pública y reglas adecuadas.
- **NAT gateway público** permite que recursos en subredes privadas inicien conexiones IPv4 hacia fuera cuando la red está configurada para ello. Se usa, por ejemplo, para descargar paquetes desde internet. Aplica costes por tiempo y datos procesados.
- **Security groups** filtran tráfico de recursos o interfaces compatibles y mantienen estado de la conexión. **Network ACLs** actúan a nivel de subred y requieren reglas para ambos sentidos del tráfico cuando corresponda.
- **VPC endpoint** da acceso privado a un servicio compatible desde la VPC. Un *gateway endpoint* a S3 puede evitar usar NAT para ese tráfico. La identidad de la aplicación y las políticas siguen controlando qué operaciones se autorizan.
- **Elastic Load Balancing (ELB)** reparte tráfico entre destinos sanos. **Application Load Balancer (ALB)** permite tomar decisiones HTTP/HTTPS como encaminar por ruta o host. **Network Load Balancer (NLB)** atiende protocolos y requisitos de capa de transporte.
- **Amazon Route 53** resuelve nombres DNS. **Amazon CloudFront** entrega contenido y puede cachearlo en el borde. Un nombre mediante Route 53 puede apuntar a CloudFront o a un balanceador, según el diseño.

Piensa en el problema de cada paso. Resolver un nombre, transportar paquetes, repartir peticiones HTTP y decidir si un usuario puede leer un objeto son decisiones distintas. Un error de DNS, un *timeout* y un `AccessDenied` sugieren puntos de partida distintos para investigar.

## Cómo se separan las aplicaciones entre sí

| Servicio | Idea | Ejemplo |
| --- | --- | --- |
| **Amazon SQS** | Cola de mensajes que guardan encargos para consumidores. | Una API registra la solicitud de un informe y un worker lo genera después. |
| **Amazon SNS** | Publicación y suscripción a un tema para distribuir una notificación. | Un evento dispara dos colas independientes para facturación y notificaciones. |
| **Amazon EventBridge** | Bus de eventos que aplica reglas y dirige eventos a destinos. | Encaminas un cambio de estado a un flujo según campos del evento. |
| **AWS Step Functions** | Orquestación explícita de pasos, decisiones, reintentos y manejo de fallos. | Validar un pedido, ejecutar un cobro y actuar según el resultado. |

Con SQS puedes **encargar**, **procesar** y **confirmar el trabajo** en instantes diferentes. Si SQS acepta el mensaje y la API responde `202 Accepted`, la API confirma que la solicitud se aceptó para procesar. El trabajo todavía puede estar pendiente. Si el consumidor está caído una hora, la cola permite reanudar el consumo al volver, dentro de la retención configurada y siempre que haya capacidad suficiente para ponerse al día.

Al recibir un mensaje, un worker dispone de un **visibility timeout**. Durante ese intervalo el mensaje queda temporalmente oculto para otros consumidores. Si el worker completa el trabajo, elimina el mensaje. Si se cae antes y vence el intervalo, el mensaje puede volver a entregarse. Una cola **Standard** puede entregar duplicados y alterar el orden, así que el sistema debe poder repetir un encargo sin producir un efecto duplicado. Esto es **idempotencia**. Una **dead-letter queue (DLQ)** sirve para aislar mensajes que fallan repetidamente y que necesitan investigación.

## Cómo sabes qué pasó

- **Amazon CloudWatch** recoge métricas y logs y permite crear alarmas. Te ayuda a comprobar si subieron la latencia o la CPU.
- **AWS CloudTrail** registra actividad de API. Te ayuda a identificar quién cambió una política y cuándo. El historial de eventos reúne eventos de administración recientes por región. Acciones sobre objetos S3, como `GetObject`, se tratan como eventos de datos y requieren configuración si quieres registrarlas.
- **AWS Config** registra cambios de configuración en recursos compatibles que monitorizas y permite evaluar reglas. Te ayuda a comprobar si un bucket cumplía las reglas de configuración cuando se evaluó.

Una alarma, una pista de auditoría y una regla de configuración son respuestas diferentes. En una incidencia puedes empezar por CloudWatch para ver el síntoma, usar CloudTrail para saber quién hizo el cambio y consultar AWS Config para ver cómo evolucionó el recurso, si ya se estaba registrando.

## Fuentes

Consultadas el 24 de noviembre de 2026.

- [Guías de decisión de AWS](https://docs.aws.amazon.com/decision-guides/)
- [Opciones de cómputo](https://docs.aws.amazon.com/decision-guides/latest/decision-guides/choosing-aws-compute-service.html), [almacenamiento](https://docs.aws.amazon.com/decision-guides/latest/decision-guides/choosing-aws-storage-service.html) y [bases de datos](https://docs.aws.amazon.com/decision-guides/latest/decision-guides/databases-on-aws-how-to-choose.html)
- [Versionado de S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/versioning-workflows.html)
- [Seguridad de red en VPC](https://docs.aws.amazon.com/vpc/latest/userguide/infrastructure-security.html), [public NAT gateways](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) y [gateway endpoints para S3](https://docs.aws.amazon.com/vpc/latest/privatelink/vpc-endpoints-s3.html)
- [Qué es Elastic Load Balancing](https://docs.aws.amazon.com/elasticloadbalancing/latest/userguide/what-is-load-balancing.html)
- [Cuándo elegir SNS, SQS o EventBridge](https://docs.aws.amazon.com/decision-guides/latest/decision-guides/sns-or-sqs-or-eventbridge.html), [colas SQS Standard](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues.html) y [visibility timeout](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-visibility-timeout.html)
- [CloudTrail y CloudWatch](https://docs.aws.amazon.com/decision-guides/latest/decision-guides/cloudtrail-or-cloudwatch.html), [eventos de S3 en CloudTrail](https://docs.aws.amazon.com/AmazonS3/latest/userguide/cloudtrail-logging-s3-info.html) y [conceptos de AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/config-concepts.html)

[Índice de estudio](../README.md) · [Lectura anterior](04-iam-y-acceso.md) · [Lectura siguiente](06-criterios-de-arquitectura.md)
