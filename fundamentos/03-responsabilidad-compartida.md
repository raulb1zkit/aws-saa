---
title: "Responsabilidad compartida"
study_order: 3
permalink: /fundamentos/03-responsabilidad-compartida.html
---

# 03. Responsabilidad compartida

**Al terminar podrás** decidir quién se ocupa de la seguridad de cada capa, cómo cambia el reparto entre EC2, RDS, Lambda y S3, y por qué todavía necesitas controlar tus propios datos y accesos.

## Seguridad de la nube y en la nube

AWS se responsabiliza de la **seguridad de la nube** (*security of the cloud*). Mantiene los centros de datos, el hardware, las redes físicas y las capas de infraestructura que hacen funcionar sus servicios. El cliente se responsabiliza de la **seguridad en la nube** (*security in the cloud*). La parte del cliente depende del servicio, su configuración y el tipo de datos.

Imagina que un comercio alquila infraestructura para una aplicación de inventario. Con EC2, AWS se encarga del host físico y la virtualización. El comercio decide qué instala, a quién deja llegar al servidor y qué actualizaciones aplica dentro del sistema invitado. Si mueve la base de datos a RDS, AWS también opera la infraestructura de esa base de datos y ejecuta las tareas de mantenimiento que gestiona el servicio. El comercio sigue decidiendo quién se conecta y cómo recuperar sus datos.

## Comparar por capas

| Servicio | Qué administra AWS | Qué debes administrar tú |
| --- | --- | --- |
| **EC2** | Hardware, virtualización y componentes de infraestructura del servicio. | Parches del sistema operativo invitado, aplicaciones, firewall y reglas de red, identidades, datos y copias. |
| **RDS** | Infraestructura, sistema anfitrión y mecanismos gestionados de mantenimiento, backups y failover según el despliegue. | Usuarios y permisos de la base, acceso de red, esquema, consultas, opciones de mantenimiento, configuración de backups y estrategia de recuperación. |
| **ECS sobre EC2** | El plano de orquestación de ECS y la infraestructura física de EC2. | Imagen del contenedor, código, configuración, identidades y las instancias que eliges administrar como capacidad del clúster. |
| **ECS sobre Fargate** | La infraestructura que ejecuta las tareas, incluidos los hosts que las alojan. | Imagen y dependencias, rol de la aplicación, CPU y memoria solicitadas, red, secretos, supervisión y datos. |
| **Lambda** | Infraestructura de ejecución y las partes del runtime que correspondan al modo y política de mantenimiento. | Código, librerías, permisos, datos, eventos de entrada, elección de runtime y política de actualización. Con imagen de contenedor te corresponde incorporar las actualizaciones a tu imagen y redesplegarla. |
| **S3** | Infraestructura de almacenamiento, operación del servicio y redundancia según la clase elegida. | Datos que subes, acceso público o privado, políticas, ciclo de vida, versionado, retención y requisitos de cifrado. Las cargas nuevas tienen cifrado de servidor por defecto. Si necesitas control sobre una clave KMS, debes elegir y configurar ese modo. |

El mantenimiento puede estar compartido por componente. En **RDS**, por ejemplo, el servicio instala ciertas actualizaciones, pero el cliente revisa la ventana de mantenimiento, las notificaciones, las opciones de versión y el impacto de una actualización en la aplicación. En **Lambda**, empaquetar tu código con una biblioteca vulnerable sigue siendo un problema de tu aplicación aun cuando AWS opere el servidor.

## Acceso, claves y secretos

**Encryption in transit** protege la comunicación. Una conexión HTTPS usa TLS entre sus extremos. Si un balanceador termina TLS y envía la petición al destino, revisa también el protocolo de ese segundo tramo. **Encryption at rest** protege datos guardados, por ejemplo en un volumen, un bucket o un snapshot. Ambas capas se configuran según qué datos y qué recorridos quieras proteger.

**AWS KMS** permite administrar y utilizar claves de cifrado. Si una organización exige una clave de cliente para un bucket, **SSE-KMS** permite a S3 usar esa clave mediante KMS. El acceso a los datos puede requerir acceso tanto al objeto como a la clave. Diseña la recuperación para que el operador autorizado pueda usar la clave cuando restaure. En cambio, **AWS Secrets Manager** almacena valores como contraseñas de base de datos y puede gestionar su rotación en configuraciones compatibles. La aplicación pide el valor del secreto con su rol. KMS se ocupa de claves y operaciones criptográficas, y Secrets Manager del ciclo de vida y acceso a valores secretos.

## Cómo analizar una incidencia

1. **Localiza la capa afectada.** ¿Es hardware, hipervisor, sistema invitado, runtime, código, configuración, identidad o datos?
2. **Identifica el servicio y modo de despliegue.** El reparto cambia entre PostgreSQL instalado en EC2 y PostgreSQL en Amazon RDS.
3. **Separa la responsabilidad del proveedor de tu plan de continuidad.** Que AWS repare un host no asegura por sí mismo que la aplicación sobreviva durante el fallo. La arquitectura decide si hay otra instancia sana, desde dónde lee datos y cómo se reanuda el tráfico.
4. **Verifica el control concreto.** Si el requisito es evitar borrados accidentales, revisa versionado y permisos de borrado. Si es recuperar en dos horas, ensaya la restauración con datos y dependencias representativos.

### Aplicaciones típicas

- Una aplicación EC2 usa un paquete de Linux vulnerable. El equipo de la aplicación administra el sistema invitado y debe parchearlo.
- Una aplicación en Fargate contiene una biblioteca obsoleta dentro de su imagen. El equipo debe reconstruir y desplegar la imagen corregida. El alojamiento gestionado desplaza el mantenimiento del host a AWS.
- Una aplicación lee documentos desde RDS tras autenticarse. AWS opera la infraestructura de RDS. El equipo administra los usuarios lógicos de PostgreSQL y sus permisos sobre tablas. Las reglas de red determinan desde dónde se puede establecer la conexión.
- Un bucket tiene cifrado en reposo, pero una identidad con acceso legítimo puede descargar datos que no necesitaba. Se revisa la autorización. Cifrar el almacenamiento y limitar la lectura cubren riesgos distintos.
- Un backup está cifrado con una clave gestionada por el cliente. El plan de recuperación debe conservar los permisos y la disponibilidad de esa clave para quien restaure.

## Qué conviene recordar

- **Of the cloud** apunta a la infraestructura y a las capas que opera AWS. **In the cloud** apunta a cómo configuras y usas el servicio.
- Para decidir de quién son los parches, ubica la capa. El hipervisor, el SO invitado y una dependencia empaquetada son componentes distintos.
- Un servicio gestionado puede reducir mantenimiento y ofrecer mecanismos de backup o failover. Tu equipo valida acceso, configuración, recuperación y objetivos de negocio.

## Fuentes

Consultadas el 24 de noviembre de 2026.

- [Modelo de responsabilidad compartida de AWS](https://aws.amazon.com/compliance/shared-responsibility-model/)
- [Responsabilidad compartida, Security Pillar](https://docs.aws.amazon.com/wellarchitected/latest/security-pillar/shared-responsibility.html)
- [Seguridad de Amazon EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security.html)
- [Uso de servicios gestionados](https://docs.aws.amazon.com/wellarchitected/2022-03-31/framework/sec_protect_compute_implement_managed_services.html)
- [AWS Fargate y AWS Lambda](https://docs.aws.amazon.com/decision-guides/latest/decision-guides/fargate-or-lambda.html) y [actualizaciones de runtime de Lambda](https://docs.aws.amazon.com/lambda/latest/dg/runtimes-update.html)
- [Cifrado por defecto de S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/default-bucket-encryption.html)
- [Protección de datos en tránsito y en reposo](https://docs.aws.amazon.com/IAM/latest/UserGuide/data-protection.html), [SSE-KMS en S3](https://docs.aws.amazon.com/AmazonS3/latest/userguide/UsingKMSEncryption.html) y [protección de datos en Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/data-protection.html)

[Índice de estudio](../README.md) · [Lectura anterior](02-infraestructura-global.md) · [Lectura siguiente](04-iam-y-acceso.md)
