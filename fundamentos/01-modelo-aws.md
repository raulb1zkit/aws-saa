---
title: "Cómo funciona AWS"
study_order: 1
permalink: /fundamentos/01-modelo-aws.html
---

# 01. Cómo funciona AWS

**Al terminar podrás** situar un recurso dentro de una cuenta y una ubicación, identificar cómo se hace una petición a AWS y leer un identificador ARN sencillo.

## Qué estás contratando

AWS ofrece recursos y operaciones que puedes solicitar mediante APIs. Una organización puede crear una máquina virtual sin comprar un servidor físico, guardar objetos sin desplegar servidores de almacenamiento o contratar una base de datos cuya operación se reparte con AWS. La parte gestionada y las unidades de facturación dependen del servicio.

Piensa en tres capas para estudiar cualquier servicio nuevo:

1. **Qué recibe y devuelve.** Una máquina a la que instalas software, un almacén de objetos al que haces peticiones o un endpoint que ejecuta código.
2. **Qué administra AWS.** Desde los centros de datos y el hipervisor hasta partes del motor, la plataforma o el escalado, según el servicio y el modo elegido.
3. **Qué eliges y pagas.** Región, capacidad, red, retención, permisos y acceso. Algunos servicios cobran por capacidad aprovisionada, otros por uso, y varios combinan ambos.

Una instancia **EC2** es el paralelo más cercano a un VPS generalista. Tú puedes instalar Linux y Docker en ella. En **S3** compras la capacidad de guardar y recuperar objetos mediante una API. Tus archivos se almacenan como objetos con una clave en un *bucket*. En **RDS** puedes seguir conectándote con el protocolo de PostgreSQL, pero AWS asume tareas de infraestructura y mantenimiento del motor. El modelo de responsabilidad se desarrolla en la lectura 03.

## Cuenta, identidad, región y recurso

| Término | Qué representa | Ejemplo |
| --- | --- | --- |
| **AWS account** | Límite administrativo y de aislamiento para identidades, recursos y facturación. Tiene un ID de 12 dígitos. | Una cuenta para producción y otra para pruebas. |
| **Principal** | Identidad que realiza la petición. | Una sesión temporal del rol `ReportReader`. |
| **Region** | Área geográfica de AWS donde se despliegan recursos regionales. | `eu-south-2` es Europe (Spain). |
| **Resource** | Entidad concreta a la que se dirige una operación. | Un bucket, una cola, una instancia o una política. |
| **Tag** | Par clave–valor asociado a un recurso compatible. | `Environment=lab` para identificarlo y, si se habilita la etiqueta de asignación, ayudar a atribuir costes. |
| **Endpoint** | Dirección de servicio a la que se envía una petición. | Un endpoint regional de EC2. El término *VPC endpoint* designa una configuración de conectividad que veremos en redes. |

Una cuenta suele poder operar en varias regiones según sus ajustes y modalidad. Dos personas pueden trabajar en la misma cuenta usando identidades distintas. Una identidad también puede asumir un rol en otra cuenta si se ha configurado el acceso. Cambiar de región cambia el ámbito de muchos recursos que ves. Cambiar de cuenta cambia el ámbito administrativo. Por ejemplo, un listado vacío en la consola puede indicar que estás en otra región o cuenta, o que el filtro de la consola oculta recursos.

Una separación por cuentas ayuda a limitar el alcance de errores y permisos. Si una automatización de laboratorio tiene un permiso amplio dentro de su cuenta, la separación reduce su alcance sobre producción siempre que no exista una delegación hacia ella. **AWS Organizations** centraliza la administración de cuentas miembro, permite agruparlas en **organizational units** (OU), consolidar facturación y aplicar políticas de control. En la lectura de IAM veremos cómo se combinan esas políticas con los permisos concretos.

**AWS Builder ID** identifica a una persona en experiencias como formación y alta por proyectos. Una sesión operativa de AWS tiene un contexto de cuenta y permisos concretos. Con la experiencia de alta simplificada, los recursos de un proyecto tienen detrás una cuenta de AWS y una región preasignada. Esto explica por qué dos tutoriales pueden mostrar pantallas de alta diferentes. El modelo de cuenta, identidad y recurso sigue sirviendo para entender ambos.

## Consola, CLI, SDK e infraestructura como código

- **AWS Management Console** es la interfaz gráfica. Es útil para orientarte, confirmar una configuración y buscar la región activa.
- **AWS CLI** convierte comandos en peticiones a las APIs de AWS. Un *profile* reúne configuración asociada a un nombre, por ejemplo región y método de acceso federado.
- **AWS SDK** es la biblioteca de tu lenguaje para llamar a esas APIs desde código. Puede cargar credenciales temporales y firmar peticiones según su entorno.
- **Infraestructura como código** describe el estado de infraestructura deseado en archivos y usa una herramienta para aplicarlo. CloudFormation es un ejemplo de AWS. La parte importante aquí es distinguir describir el estado, aplicar cambios y verificar qué ha quedado creado.

Al leer un comando, acostúmbrate a identificar **a qué servicio llama, qué acción realiza, en qué región y como qué identidad**. Las variables, perfiles y opciones explícitas pueden cambiar el resultado. Con una sesión configurada, `aws sts get-caller-identity` devuelve la cuenta, el ARN y el identificador del llamante. Es una operación de consulta. Su salida no contiene claves secretas, aunque puede mostrar identificadores que conviene ocultar al compartir capturas.

También distinguirás **control plane**, el plano que crea o configura recursos, y **data plane**, el plano que atiende el tráfico de uso. Configurar el versionado de un bucket es una operación de administración. Descargar un objeto es uso de datos. La distinción ayuda al diagnosticar fallos. Según el servicio y el incidente, los recursos ya aprovisionados pueden seguir atendiendo peticiones aunque ciertas operaciones de control estén degradadas.

## ARN y permisos

Un **Amazon Resource Name** identifica un recurso en contextos como políticas IAM. Una forma común es `arn:partition:service:region:account-id:resource`. El formato y los campos presentes dependen del servicio.

| ARN ilustrativo | Qué identifica |
| --- | --- |
| `arn:aws:iam::111122223333:role/ReportReader` | Un rol IAM. El campo región está vacío. |
| `arn:aws:ec2:eu-south-2:111122223333:instance/i-0123456789abcdef0` | Una instancia en una cuenta y región concretas. |
| `arn:aws:s3:::ejemplo-formacion` | Un bucket S3 de propósito general. |
| `arn:aws:s3:::ejemplo-formacion/reports/*` | Un patrón de ARN para objetos cuyas claves empiezan por `reports/`. |

En una política de S3, el ARN del bucket y el ARN de sus objetos representan recursos distintos. `s3:ListBucket` actúa sobre el bucket. `s3:GetObject` actúa sobre los objetos. Que un ARN de S3 omita región y cuenta en su sintaxis **no determina dónde se almacenan los datos**. La ubicación del bucket se decide aparte.

## Qué conviene recordar

- Para saber dónde buscar un recurso, pregunta por **cuenta y región**. Para saber quién puede operarlo, pregunta por **principal, acción y recurso**.
- Un servicio gestionado desplaza trabajo operativo. La arquitectura todavía debe decidir acceso, configuración, datos y comportamiento ante fallos.
- Si un caso menciona un ARN, reconoce su servicio, recurso y ámbito. En el examen importa más recoger la intención del permiso que memorizar formatos de todos los servicios.

## Fuentes

Consultadas el 24 de noviembre de 2026.

- [Qué es una cuenta AWS](https://docs.aws.amazon.com/accounts/latest/reference/)
- [Aislamiento con varias cuentas](https://docs.aws.amazon.com/accounts/latest/reference/welcome-multiple-accounts.html)
- [Comparación de opciones de alta](https://docs.aws.amazon.com/accounts/latest/reference/sign-up-for-aws.html)
- [Configuración y perfiles de AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) y [GetCallerIdentity](https://docs.aws.amazon.com/STS/latest/APIReference/API_GetCallerIdentity.html)
- [Referencia de ARN](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference-arns.html)
- [Límites de aislamiento de fallos y planos de AWS](https://docs.aws.amazon.com/whitepapers/latest/aws-fault-isolation-boundaries/welcome.html)

[Índice de estudio](../README.md) · [Lectura siguiente](02-infraestructura-global.md)
