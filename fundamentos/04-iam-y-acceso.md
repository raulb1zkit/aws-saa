---
title: "IAM y acceso"
study_order: 4
permalink: /fundamentos/04-iam-y-acceso.html
---

# 04. IAM, identidades y acceso

**Al terminar podrás** elegir entre acceso humano federado y roles de aplicación, interpretar una política sencilla y explicar cómo conviven los permisos con la conectividad.

## De una identidad a una petición autorizada

**AWS Identity and Access Management (IAM)** permite controlar quién puede realizar acciones sobre recursos de AWS. Una petición firmada pasa por dos preguntas. La **autenticación** determina quién la hizo. La **autorización** decide si esa identidad puede ejecutar la acción sobre ese recurso en ese contexto. Por ejemplo, una credencial temporal válida permite a AWS identificar una sesión de rol. La sesión todavía necesita autorización para leer el objeto solicitado.

Para estudiar una petición, escribe **principal → action → resource → context**. Un ejemplo sería «la sesión del rol `ReportReader` intenta `s3:GetObject` sobre `reports/noviembre.csv` en el bucket `ejemplo-formacion`». El contexto puede incluir condiciones sobre la petición o elementos de la identidad y del recurso. Un fallo DNS, de ruta o de firewall ocurre en otra capa, aunque veas que ambos problemas impiden usar la aplicación.

## Quién se conecta y con qué credenciales

| Identidad o mecanismo | Qué aporta | Cuándo reconocerlo |
| --- | --- | --- |
| **AWS account root user** | Identidad de nivel de cuenta en el modelo tradicional. Algunas tareas de cuenta requieren root. | Protege el acceso de root con MFA y evita claves de acceso de root. Usa acceso administrativo delegado para el trabajo diario. |
| **IAM user** | Identidad persistente de una cuenta. Puede tener credenciales de larga duración. | Integraciones que requieren este mecanismo tras verificar si admiten uno temporal. Para acceso humano, el patrón habitual de AWS es federación. |
| **IAM group** | Colección de usuarios IAM para asignarles permisos conjuntamente. | Administración de usuarios IAM existentes. Un grupo IAM no firma peticiones y solo agrupa usuarios IAM. |
| **IAM role** | Identidad asumible que proporciona permisos mediante una sesión temporal. | Aplicaciones en EC2, tareas ECS, funciones Lambda, pipelines y acceso entre cuentas. |
| **IAM Identity Center** | Acceso humano centralizado y federado a cuentas. Los *permission sets* determinan los permisos del acceso asignado. | Administrar el acceso de personas a una o varias cuentas. |
| **AWS STS** | Emite credenciales temporales formadas por un par de claves y un token de sesión, con caducidad. | `AssumeRole`, federación y aplicaciones que reciben credenciales efímeras. |

**MFA** (*multi-factor authentication*) añade un factor al inicio de sesión. **Least privilege** limita las acciones, recursos y condiciones al trabajo que se necesita. Son controles complementarios. Una identidad con MFA todavía puede tener permisos excesivos.

## Las dos políticas de un rol

- **Trust policy.** Es una política basada en recurso vinculada al rol. Establece qué principal puede asumirlo y bajo qué condiciones. Para EC2, el principal de servicio habitual es `ec2.amazonaws.com`. Con federación, la confianza puede depender de atributos de quien presenta el token.
- **Permissions policy.** Describe qué acciones permite el rol después de asumirlo. Por ejemplo, la aplicación puede leer objetos de un prefijo concreto de S3 pero no borrarlos.

En **EC2**, el *instance profile* asocia el rol a la instancia. El SDK de una aplicación puede obtener las credenciales temporales a través de la infraestructura de metadatos de EC2. En **ECS**, el **task role** da permisos al código que corre dentro del contenedor. El **task execution role** cubre las operaciones que realiza el agente de ECS en nombre de la tarea, como obtener una imagen privada o enviar logs según la configuración. En **Lambda**, el **execution role** da a la función permisos para acceder a servicios. Cuando S3 debe poder invocar una función, se autoriza esa invocación por separado en la política basada en recurso de la función.

Un pipeline de **GitHub Actions** puede autenticarse mediante **OIDC**. GitHub emite un token, AWS lo valida frente a la confianza del rol y emite credenciales temporales. La confianza se restringe al repositorio y a una rama o entorno de despliegue concreto, según el flujo. La política de permisos define lo que puede desplegar el pipeline. La confianza decide quién entra y los permisos, qué puede hacer una vez dentro.

## Cómo leer una política JSON

Esta política de identidad de ejemplo autoriza leer objetos del prefijo `reports/` de un bucket ficticio. El permiso se concedería al usuario o rol al que se adjunte. El nombre del bucket es ilustrativo y el ejemplo se estudia en papel.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ReadReports",
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::ejemplo-formacion/reports/*"
    }
  ]
}
```

- `Effect` indica si la declaración permite o deniega. `Action` identifica la operación y `Resource` acota a qué recurso se aplica.
- `Condition` permite acotar el contexto cuando la acción admite la clave de condición.
- `Principal` aparece cuando la declaración basada en recurso necesita identificar a quién se aplica. En una política de identidad, el destino de la propia política determina su identidad.
- `Version` indica la versión del lenguaje de políticas. `Sid` ayuda a identificar una declaración.
- `s3:GetObject` permite recuperar un objeto por su clave. Para listar, se requiere `s3:ListBucket` en el **ARN del bucket**, y se puede limitar la petición con la condición `s3:prefix`. Para obtener una versión concreta, revisa `s3:GetObjectVersion`. Cada acción tiene su propio alcance.

En S3, un prefijo es parte de la clave. La consola puede mostrarlo como una carpeta. En una política, la distinción entre bucket, patrón de objetos y condición de listado es importante.

## De la política al resultado

Empieza con **implicit deny**. Una petición necesita una autorización aplicable. Si una política aplicable contiene un **explicit deny**, esa denegación gana a cualquier permiso. Dentro de una cuenta, una política de identidad o de recurso puede contribuir a autorizar muchos accesos. Hay matices según el principal y el recurso. Las políticas de claves **KMS** y de confianza de roles siguen reglas particulares, así que revisa su modelo antes de generalizar.

Para acceso entre cuentas, revisa normalmente la autorización en la cuenta del principal y la autorización o confianza en la cuenta del recurso. Si el mecanismo es asumir un rol de la otra cuenta, revisa el permiso para `sts:AssumeRole`, la confianza del rol destino y luego los permisos de la nueva sesión. Para una clave KMS usada desde otra cuenta, comprueba tanto la política de clave como los permisos del principal.

**Service control policies (SCPs)** de Organizations limitan los permisos máximos de los principales en cuentas miembro, incluido root de una cuenta miembro dentro de su alcance. Por sí solas no conceden permisos. **Resource control policies (RCPs)** imponen límites sobre recursos compatibles en cuentas miembro. Una **permissions boundary** acota el máximo que una política de identidad puede otorgar a un usuario o rol. Para el examen, reconoce qué capa pone el límite y cuál expresa el permiso. Las SCP no se aplican a los principales de la cuenta de administración de la organización.

## Caso práctico de razonamiento

Un servicio lee informes desde una instancia privada. Cuenta con un *gateway endpoint* de S3. Ante un `AccessDenied`, comprobaría el ARN del principal en uso, la acción, el prefijo, los permisos del bucket y del endpoint, y las denegaciones aplicables. Si el objeto usa **SSE-KMS**, también revisaría que el acceso a la clave permita descifrar cuando corresponda. Ante un *timeout*, examinaría primero ruta, DNS y conectividad. Un timeout ofrece poca información sobre la evaluación de IAM porque quizá la solicitud ni siquiera llegó a S3.

Para investigar quién cambió una política, piensa en **CloudTrail**. Para encontrar accesos concedidos a principales externos o revisar políticas, reconoce **IAM Access Analyzer**. Para asignar acceso laboral de personas a varias cuentas con permisos consistentes, piensa en **IAM Identity Center**.

## Fuentes

Consultadas el 24 de noviembre de 2026.

- [Buenas prácticas de IAM](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html) y [protección de root](https://docs.aws.amazon.com/IAM/latest/UserGuide/root-user-best-practices.html)
- [Conceptos de IAM y contexto de autorización](https://docs.aws.amazon.com/IAM/latest/UserGuide/intro-structure.html) y [evaluación de políticas](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html)
- [Evaluación entre cuentas](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic-cross-account.html) y [políticas de Organizations](https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_scps.html)
- [Credenciales temporales STS](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_request.html), [roles de EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html) y [permission sets de Identity Center](https://docs.aws.amazon.com/singlesignon/latest/userguide/permissionsetsconcept.html)
- [Rol de aplicación ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-iam-roles.html) y [rol de ejecución ECS](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_execution_IAM_role.html)
- [OIDC de GitHub Actions en AWS](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-aws)
- [Acciones, recursos y condiciones de S3](https://docs.aws.amazon.com/service-authorization/latest/reference/list_s3.html)

[Comprobar este tema con el test 04](test-04.md) · [Todos los tests](tests.md)

[Índice de estudio](../README.md) · [Lectura anterior](03-responsabilidad-compartida.md) · [Lectura siguiente](05-mapa-de-servicios.md)
