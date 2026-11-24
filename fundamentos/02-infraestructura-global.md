---
title: "Regiones, AZ y edge"
study_order: 2
permalink: /fundamentos/02-infraestructura-global.html
---

# 02. Regiones, zonas de disponibilidad y edge

**Al terminar podrás** distinguir *Region*, *Availability Zone* y *edge location*, situar recursos habituales y razonar sobre el fallo de una zona o una región.

## Tres escalas de infraestructura

| Término | Qué es | Decisión que resuelve |
| --- | --- | --- |
| **AWS Region** | Área geográfica separada que agrupa varias AZ. Ejemplos de códigos son `eu-south-2` (España) y `eu-west-1` (Irlanda). | Dónde desplegar los recursos, qué latencia esperar, dónde deben residir los datos y qué servicios están disponibles. |
| **Availability Zone (AZ)** | Grupo aislado de uno o más centros de datos dentro de una región. Las AZ de la región se conectan mediante redes de baja latencia. | Cómo aislar fallos de infraestructura dentro de una región. |
| **Edge location / point of presence (PoP)** | Punto de la red perimetral de AWS que puede atender tráfico de servicios como CloudFront. | Cómo mejorar la entrega a usuarios distribuidos y evitar viajes innecesarios al origen. |

En una región se pueden desplegar componentes en varias AZ. AWS diseña las zonas para aislar fallos y ofrece conectividad entre ellas. La aplicación necesita usar esa separación. Dos máquinas en subredes distintas de la **misma** AZ siguen compartiendo el riesgo de esa AZ. Si están en dos AZ pero dependen de una base de datos en una sola zona, la capa web tiene redundancia y la ruta completa todavía tiene un punto de fallo.

En diagramas verás nombres como `eu-west-1a`. Al coordinar un despliegue entre cuentas, si necesitas identificar la misma zona física, usa el **AZ ID**, como `euw1-az1`. El nombre visible de la zona puede tener asignaciones diferentes según la región y cuándo se creó la cuenta. Para el examen suele bastar con entender el dominio de fallo.

## Ámbito de recursos frecuentes

| Ámbito práctico | Recursos o servicios | Qué implica |
| --- | --- | --- |
| **Una AZ** | Instancia EC2, volumen EBS, subred de una VPC. | El volumen EBS se conecta a una instancia de su misma AZ. Para recrear capacidad en otra zona se puede crear otro volumen desde un snapshot accesible en la región. |
| **Una región** | VPC, colas SQS, un bucket S3, bases de datos RDS. | Una VPC puede tener subredes en varias AZ. Que RDS sea un servicio regional admite configuraciones distintas como Single-AZ o Multi-AZ. Que una clase de S3 replique entre varias AZ no cambia la región del bucket. |
| **Alcance global o de borde** | IAM, DNS público de Route 53, distribución CloudFront. | IAM administra identidades para la cuenta en su partición. Route 53 responde consultas DNS y CloudFront entrega contenido cerca del usuario. Los orígenes pueden estar en una región, o fuera de AWS. |

El ámbito de la API y el de un recurso pueden diferir. Un servicio puede tener API regional y permitir una configuración global o multirregión. Por ejemplo, el nombre de un bucket de propósito general creado en el espacio de nombres global es único en su partición. Los datos de ese bucket se guardan en la región elegida salvo que configures una transferencia o replicación. Para una clase de almacenamiento, comprueba su diseño concreto. **S3 Standard** almacena redundancia en varias AZ de la región y **S3 One Zone-IA** la limita a una AZ.

## Cómo decidir una región

1. **Residencia de datos y restricciones de cumplimiento.** Si los datos deben permanecer en una geografía, incluye réplicas, backups, logs y copias de analítica en la decisión. Una región cercana a tus usuarios puede no cumplir ese requisito.
2. **Latencia y conectividad.** Mide a los usuarios, la conexión de tu oficina y la latencia entre dependencias. A veces es más importante colocar cerca API y base de datos que colocar la API cerca de todos los clientes a la vez.
3. **Servicios, capacidad y precios.** Comprueba que las funciones y tipos de recursos están disponibles en la región elegida. El coste puede diferir por región, zona y tráfico.
4. **Recuperación e integración.** Si la organización requiere recuperarse ante la pérdida de una región, planifica otra ubicación, replicación, encaminamiento, permisos y procedimientos. Valida también la capacidad allí.

Una web de documentación europea podría tener un único origen regional y usar un **CDN** para lectores fuera de Europa. **Route 53** puede resolver el nombre público a una distribución **CloudFront**. El navegador pide el objeto al PoP. En un *cache hit*, el PoP puede responder con la copia en caché. En un *cache miss*, la distribución obtiene el contenido del origen. Ajustar la caché reduce consultas al origen y latencia. La base de datos y su recuperación siguen teniendo sus propios requisitos. Si has trabajado con Cloudflare, la analogía útil es distinguir resolución DNS, capa de entrega y servidor de origen.

## Qué significa sobrevivir a un fallo

- **Cae una instancia.** Puedes dirigir tráfico a otra instancia sana y reponer la que falló. Necesitas capacidad y un mecanismo que detecte el fallo.
- **Cae una AZ.** El camino crítico debe seguir operativo en otra AZ, desde la entrada hasta el acceso a datos. El estado que solo existía en la zona afectada puede requerir recuperación.
- **Cae una región.** Un diseño Multi-AZ dentro de esa región comparte la dependencia regional. Un diseño de recuperación en otra región necesita datos allí, capacidad, un mecanismo para enviar tráfico y pruebas de restauración o failover.
- **Se borra un dato por error.** La replicación puede propagar el borrado según su configuración. Las versiones, snapshots o copias recuperables cubren otro tipo de fallo. Verifica que puedes acceder y restaurarlos.

Otros nombres aparecen al estudiar AWS global. **Local Zones** acercan parte de la infraestructura regional a algunas áreas metropolitanas. **Wavelength** lleva servicios concretos cerca de redes de operadores compatibles. **Outposts** extiende ciertos servicios a instalaciones del cliente. Para fundamentos basta reconocer el motivo de cada opción. Una *edge location* de CloudFront se usa como punto de entrega, con un papel distinto al de desplegar cómputo o datos en una AZ.

## Ejemplo de razonamiento

Una empresa pide seguir aceptando pedidos cuando se pierde una AZ y tiene un presupuesto limitado. Un primer diseño es una sola región con un balanceador y servidores web en dos AZ, más una base de datos configurada para failover entre zonas. Comprueba que la AZ restante sostiene los pedidos y que los servidores web reconectan a la base de datos. El presupuesto y el requisito de fallo guían la elección. Un segundo despliegue regional cobraría sentido si también se exige recuperación ante un incidente de región.

## Fuentes

Consultadas el 24 de noviembre de 2026.

- [Regiones y zonas de disponibilidad](https://docs.aws.amazon.com/global-infrastructure/latest/regions/aws-regions-availability-zones.html)
- [AZ IDs para recursos de AWS](https://docs.aws.amazon.com/ram/latest/userguide/working-with-az-ids.html)
- [Límites de aislamiento de fallos de AWS](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/welcome.html)
- [Subredes de Amazon VPC](https://docs.aws.amazon.com/vpc/latest/userguide/configure-subnets.html) y [volúmenes EBS](https://docs.aws.amazon.com/ebs/latest/userguide/ebs-volumes.html)
- [Protección y redundancia de datos en S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/DataDurability.html) y [reglas de nombres de buckets S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html)
- [Entrega de contenido de CloudFront](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/HowCloudFrontWorks.html)
- [Well-Architected sobre despliegues en varias ubicaciones](https://docs.aws.amazon.com/wellarchitected/latest/framework/rel10-bp01.html)
- [Opciones de cómputo de AWS](https://docs.aws.amazon.com/decision-guides/latest/decision-guides/choosing-aws-compute-service.html)

[Índice de estudio](../README.md) · [Lectura anterior](01-modelo-aws.md) · [Lectura siguiente](03-responsabilidad-compartida.md)
