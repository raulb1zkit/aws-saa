---
title: "Facturación y prácticas"
study_order: 7
permalink: /fundamentos/07-costes-y-entornos.html
---

# 07. Facturación, Free Tier y entornos de práctica

**Al terminar podrás** reconocer qué genera costes en un diseño, comprobar el contexto de una práctica y elegir entre cuenta propia, laboratorio gestionado y emulador. Esta lectura incluye condiciones de producto que pueden cambiar. Fecha de verificación **24 de noviembre de 2026**.

## De dónde sale un cargo

Un diseño puede combinar recursos por hora, peticiones, almacenamiento, transferencia y compras de compromiso. Al comparar dos arquitecturas, añade los componentes que necesita cada una para funcionar y para recuperarse.

| Componente | Qué buscar en el precio |
| --- | --- |
| **EC2, RDS y balanceadores** | Tiempo o capacidad consumida. El motor o tipo de recurso puede añadir almacenamiento, I/O, licencias o unidades de carga. |
| **EBS y snapshots** | Capacidad aprovisionada, funciones adicionales y datos conservados. Un volumen puede permanecer después de parar o terminar una instancia. |
| **S3** | Datos guardados, versiones anteriores, solicitudes, recuperación y transferencia aplicable. Las clases de archivo pueden introducir tiempos de recuperación y cobros por permanencia mínima. |
| **NAT gateway, endpoints e IPv4 públicas** | Horas y datos procesados según el recurso. Un *gateway endpoint* de S3 no tiene cargo adicional por el endpoint. Los endpoints de interfaz tienen su propio modelo de precio. |
| **CloudWatch, DNS y KMS** | Métricas o logs según volumen y retención, zonas y consultas DNS, claves y operaciones según el modo elegido. Son servicios auxiliares que pueden sobrevivir al recurso principal. |

**On-Demand** es un punto de comparación cuando la demanda o su duración son inciertas. **Savings Plans** ofrecen descuentos a cambio de un compromiso de gasto de cómputo elegible. **Reserved Instances** son compromisos o beneficios de facturación según el servicio y sus condiciones. **Spot** permite usar capacidad disponible de EC2 a menor precio, con posibilidad de interrupción. Una pregunta con trabajos reintentables apunta a evaluar Spot. Una base crítica con exigencia de continuidad se estudia a partir de sus requisitos de servicio y recuperación. Los compromisos de compra se consideran después de medir una demanda base y comprobar su elegibilidad.

**Ejemplo.** Dos instancias privadas envían lotes a S3. Con NAT, en la factura pueden aparecer horas y procesamiento del gateway además del uso de S3. Si todo ese tráfico va a S3 dentro de la región y la arquitectura admite un *gateway endpoint*, el endpoint puede quitar el salto por NAT. Si las instancias también necesitan descargar paquetes públicos u otros servicios, ese tráfico requiere otra ruta. Un cambio reduce los costes del NAT en la medida en que reduzca su uso o permita retirar el gateway.

## Free Tier y créditos

A fecha de consulta, AWS ofrece para clientes nuevos elegibles **100 USD iniciales** y hasta **100 USD adicionales** al completar actividades aprobadas. La oferta depende de la elegibilidad de la cuenta. Haber tenido una cuenta de AWS puede excluirte.

- **Free plan.** Permite usar un conjunto limitado de servicios y funciones durante hasta seis meses o hasta agotar los créditos, lo que ocurra antes. La cuenta o el proyecto se suspende según las condiciones del plan cuando se alcanza el límite. Comprueba el proceso de alta antes de activar funciones avanzadas o entrar en una organización, porque ciertas acciones cambian el plan.
- **Paid plan.** Permite facturar por uso. Los créditos elegibles cubren determinados cargos hasta que se agotan o vencen. Pagas por el uso que quede fuera de la cobertura. Las ofertas Always Free y las pruebas tienen límites propios.
- **Cuentas anteriores al 15 de julio de 2025.** Siguen el programa anterior, con ofertas y caducidades propias. La ayuda sobre seis meses y hasta 200 USD describe el programa nuevo.
- **Experiencia por proyectos.** El alta simplificada de 2026 configura cuenta y región de proyecto. Algunas funciones granulares de IAM, red o multirregión requieren opciones avanzadas. AWS tiene también un alta avanzada, que ofrece otro nivel de control. La consola correspondiente determina dónde revisar plan, saldo, restricciones y fecha de expiración.

**Antes de usar la cuenta**, comprueba el tipo de plan, el saldo, la caducidad por crédito, la cobertura, la región y si la cuenta ya presenta cargos. En la consola habitual mira **Billing and Cost Management → Credits, Free Tier y Bills**. En la experiencia por proyectos mira **AWS Settings → Billing** y el proyecto concreto. Usa un perfil que sepas identificar. Un Builder ID de formación y un acceso a una cuenta con recursos son contextos diferentes.

## Presupuesto, estimación y coste observado

| Herramienta | Pregunta que responde |
| --- | --- |
| **AWS Pricing Calculator** | ¿Cuánto valdría este escenario antes de crearlo? Introduce región, horas, tráfico y tamaño. El tratamiento de descuentos y Free Tier depende de la estimación y de sus supuestos. |
| **AWS Budgets** | ¿Qué coste o uso real o previsto se acerca a un umbral? Puedes enviar avisos a uno o más destinatarios. Algunas configuraciones admiten acciones concretas. |
| **AWS Cost Explorer** | ¿Qué servicios, regiones o tipos de uso explican la evolución de mis costes? |
| **Bills** | ¿Qué conceptos aparecen en la facturación y qué créditos se han aplicado? |

**Un aviso de AWS Budgets no es un límite de gasto.** Los costes pueden tardar en aparecer y se pueden seguir acumulando después de superar el umbral. En una cuenta que admite Budgets, un ejemplo de laboratorio es usar un presupuesto mensual de 5 USD con avisos reales al 50 % y al 100 %. Puedes añadir una alerta de previsión. Si quieres observar el consumo bruto incluso cuando se cubre con créditos, verifica que la vista de costes excluya dichos créditos. La plantilla *Zero spend* permite vigilar cuándo aparece gasto según sus parámetros. Confirma siempre la identidad con permiso para acceder a facturación.

En ciertas cuentas de la experiencia nueva, el **spend limit** de un proyecto Paid es otro mecanismo. AWS lo describe como un límite de coste mensual antes de impuestos, con un mínimo de **20 USD o más** según la estimación de uso. Al alcanzarlo, AWS pausa el proyecto según sus condiciones. Comprueba la disponibilidad y alcance en el proyecto concreto. Un límite de proyecto y una alerta de presupuesto sirven para controles diferentes.

## Cómo elegir dónde practicar

| Entorno | Para qué sirve | Alcance |
| --- | --- | --- |
| **Floci en Docker** | Operaciones locales con S3 y SQS, uso de CLI o SDK, versionado de objetos y lógica de una aplicación con una cola. | Proyecto MIT con un endpoint local. Documenta la cobertura de sus APIs. Sus resultados prueban el flujo local. IAM, red, aislamiento de AWS y facturación requieren verificaciones propias. |
| **AWS Educate** | Introducciones guiadas a S3, EC2, VPC y RDS. | AWS publica laboratorios gratuitos con registro por correo sin tarjeta. El catálogo, los permisos y lo que puedes crear dependen del ejercicio. |
| **AWS Academy Learner Lab** | Prácticas de clase con un entorno gestionado por el programa. | Requiere centro participante y matriculación. Pregunta al centro por saldo, caducidad y servicios habilitados. |
| **AWS Skill Builder y workshops de evento** | Laboratorios guiados y escenarios de práctica con infraestructura temporal cuando se incluye. | Skill Builder mezcla recursos gratuitos y de suscripción. En los workshops, confirma si el evento proporciona sandbox. Si sigues instrucciones públicas con cuenta propia, los recursos se imputan a esa cuenta. |
| **Cuenta personal de AWS** | Comprobar comportamiento de AWS para servicios, topologías y configuraciones disponibles en la cuenta. | Tú verificas quién paga, región, costes, dependencias y limpieza. |

Para un primer ensayo de S3 basta subir un fichero ficticio dos veces con la misma clave, ver las versiones y recuperar la copia tras el borrado. Floci sirve para ese flujo, con bucket y contenido descartables. Si la pregunta del ejercicio es si una política o una ruta funcionaría en AWS, prepara una verificación en un entorno real autorizado. El emulador documenta, por ejemplo, que su filtrado opcional de security groups en contenedores no implementa enrutamiento de VPC, NACL, NAT Gateway ni peering.

## Cómo comprobar el cierre de una práctica

1. **Antes.** Identifica cuenta, región, plan, recursos y duración. Consulta precios y decide qué resultado demuestra la práctica. Usa contenido de prueba. Si el ejercicio usa infraestructura propia, planifica qué borrar en ese mismo momento.
2. **Durante.** Lleva un inventario pequeño de recursos y dependencias. Las etiquetas de laboratorio ayudan cuando el servicio las admite. Distingue servicios que sólo consultas de recursos que aprovisionas.
3. **Al cerrar.** Elimina los recursos del laboratorio conforme al orden de dependencias y verifica cada consola o listado. Si hay Auto Scaling, desactiva o elimina la configuración que recrearía instancias. Revisa NAT, balanceador, endpoints de interfaz, IPv4, volúmenes, snapshots, bases, backups y logs si los creaste.
4. **S3 con versionado.** Comprueba que el bucket es el de prueba. Vacíalo incluyendo **todas las versiones y delete markers**, después borra el bucket. El borrado corriente de un objeto con versionado puede ocultarlo y dejar cobrando el almacenamiento de las versiones.
5. **Después.** Comprueba si quedan dependencias en otras regiones donde trabajaste y vuelve a revisar Bills cuando el consumo se refleje. Anota coste estimado, coste observado y cómo confirmaste la limpieza.

## Fuentes

- [FAQ actual de AWS Free Tier](https://aws.amazon.com/free/free-tier-faqs/), [planes de la experiencia avanzada](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier-plans.html) e [introducción del programa y Free Tier legado](https://aws.amazon.com/blogs/aws/aws-free-tier-update-new-customers-can-get-started-and-explore-aws-with-up-to-200-in-credits/)
- [Comparación de altas](https://docs.aws.amazon.com/accounts/latest/reference/sign-up-for-aws.html), [lectura de Billing en AWS Settings](https://docs.aws.amazon.com/accounts/latest/reference/view-billing-information.html) y [créditos en AWS Billing](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/useconsolidatedbilling-credits.html)
- [AWS Pricing Calculator](https://docs.aws.amazon.com/pricing-calculator/latest/userguide/getting-started.html), [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html) y [spend limits por proyecto](https://docs.aws.amazon.com/accounts/latest/reference/create-spend-limit.html)
- [Precios de Amazon S3](https://aws.amazon.com/s3/pricing/), [precios de Amazon VPC](https://aws.amazon.com/vpc/pricing/) y [opciones de compra de EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instance-purchasing-options.html)
- [Cargos inesperados](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/checklistforunwantedcharges.html) y [vaciado de un bucket de propósito general](https://docs.aws.amazon.com/AmazonS3/latest/userguide/empty-bucket.html)
- [AWS Educate](https://aws.amazon.com/education/awseducate/), [AWS Academy](https://aws.amazon.com/training/awsacademy/), [AWS Skill Builder](https://aws.amazon.com/training/digital/) y [AWS Workshop Studio](https://catalog.workshops.aws/)
- [Repositorio oficial de Floci](https://github.com/floci-io/floci) y [cobertura de S3 en Floci](https://floci.io/floci/services/s3/)

[Comprobar este tema con el test 07](test-07.md) · [Todos los tests](tests.md)

[Índice de estudio](../README.md) · [Lectura anterior](06-criterios-de-arquitectura.md)
