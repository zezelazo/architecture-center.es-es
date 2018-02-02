---
title: Fundamentos de calidad del software
description: "Describe cinco fundamentos de calidad del software: la escalabilidad, la disponibilidad, la resistencia, la administración y la seguridad."
author: MikeWasson
ms.openlocfilehash: 1d5e30602cafa0d39f92de3101974e77ae258595
ms.sourcegitcommit: a7aae13569e165d4e768ce0aaaac154ba612934f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 01/30/2018
---
# <a name="pillars-of-software-quality"></a>Fundamentos de calidad del software 

Una aplicación correcta en la nube se centrará en estos cinco fundamentos de calidad del software: la escalabilidad, la disponibilidad, la resistencia, la administración y la seguridad.

| Fundamento | DESCRIPCIÓN |
|--------|-------------|
| Escalabilidad | La capacidad de un sistema para controlar el aumento de la carga. |
| Disponibilidad | La proporción de tiempo que un sistema es funcional y está en funcionamiento. |
| Resistencia | La capacidad de un sistema de recuperarse de los errores y seguir funcionando. |
| Administración | Procesos de operaciones que mantienen un sistema ejecutándose en producción. |
| Seguridad | Protección de las aplicaciones y los datos frente a amenazas. |

## <a name="scalability"></a>Escalabilidad

La escalabilidad es la capacidad de un sistema para controlar el aumento de la carga. Una aplicación tiene fundamentalmente dos maneras de escalar. El escalado *vertical* implica el aumento de la capacidad de un recurso, por ejemplo mediante el uso de una máquina virtual de mayor tamaño. El escalado *horizontal* consiste en agregar nuevas instancias de un recurso, como máquinas virtuales o réplicas de base de datos. 

El escalado horizontal aporta ventajas considerables sobre el escalado vertical:

- Auténtico escalado en la nube. Las aplicaciones pueden diseñarse para ejecutarse en cientos o incluso miles de nodos, alcanzando escalados que no son posibles en un único nodo.
- El escalado horizontal es elástico. Puede agregar más instancias si aumenta la carga, o quitarlas durante períodos más tranquilos.
- El escalado horizontal se puede activar automáticamente, según una programación o en respuesta a cambios en la carga. 
- El escalado horizontal puede ser más económico que el escalado vertical. La ejecución de varias máquinas virtuales pequeñas puede costar menos que una sola máquina virtual grande. 
- El escalado horizontal también puede mejorar la resistencia, ya que agrega redundancia. Si una instancia se queda inactiva, la aplicación sigue funcionando.

Una ventaja del escalado vertical es que puede llevarlo a cabo sin necesidad de efectuar cambios en la aplicación. Sin embargo, en algún momento alcanzará un límite en el que no podrá escalar verticalmente nada más. En ese momento, cualquier escalado habrá de ser horizontal. 

El escalado horizontal debe estar diseñado en el sistema. Por ejemplo, puede escalar horizontalmente las máquinas virtuales colocándolas detrás de un equilibrador de carga. Sin embargo, cada máquina virtual del grupo debe ser capaz de controlar las solicitudes de cliente, por lo que la aplicación debe ser sin estado o almacenar el estado externamente (por ejemplo, en una memoria caché distribuida). Los servicios PaaS administrados suelen tienen el escalado horizontal y el escalado vertical automático integrados. La facilidad del escalado en estos servicios es una importante ventaja a la hora de usar los servicios de PaaS.

Sin embargo, el mero hecho de agregar más instancias no implicará el escalado de una aplicación. Quizá simplemente traslade el cuello de botella a alguna otra parte. Por ejemplo, si escala un front-end web para administrar más solicitudes de cliente, podría desencadenar contenciones de bloqueo en la base de datos. En este caso, tendría que considerar la posibilidad de tomar medidas adicionales, como la simultaneidad optimista o la partición de datos, para habilitar el mejor rendimiento de la base de datos.

Realice siempre pruebas de carga y rendimiento para detectar estos posibles cuellos de botella. Los elementos con estado de un sistema, como las bases de datos, son la causa más común de los cuellos de botella, y requieren un cuidadoso diseño para su escalado horizontal. La resolución de un cuello de botella puede revelar otros cuellos de botella en otros lugares.

Use la [lista de comprobación de escalabilidad][scalability-checklist] para revisar el diseño desde un punto de vista de la escalabilidad.

### <a name="scalability-guidance"></a>Guía de escalabilidad

- [Design patterns for scalability and performance][scalability-patterns] (Patrones de diseño para la escalabilidad y el rendimiento)
- Prácticas recomendadas: [Autoscaling][autoscale] (Escalado automático), [Background jobs][background-jobs] (Trabajos en segundo plano), [Caching][caching] (Almacenamiento en caché), [CDN][cdn] (Red CDN), [Data partitioning][data-partitioning] (Creación de particiones para datos).

## <a name="availability"></a>Disponibilidad

La disponibilidad es la proporción de tiempo que un sistema es funcional y está en funcionamiento. Normalmente se mide como porcentaje del tiempo de actividad. Los errores de aplicación, los problemas de infraestructura y la carga del sistema pueden reducir la disponibilidad. 

Una aplicación en la nube debe tener un objetivo de nivel de servicio que defina claramente la disponibilidad esperada y la manera de medir la disponibilidad. Al definir la disponibilidad, examine la ruta crítica. El front-end web debería ser capaz de atender las solicitudes de cliente, pero si se produce un error en cada transacción porque no se puede conectar con la base de datos, la aplicación no estará disponible para los usuarios. 

La disponibilidad a menudo se describe en términos de la cantidad de "9"; por ejemplo, "cuatro 9" significa un tiempo de actividad del 99,99 %. En la tabla siguiente se muestra el tiempo de inactividad acumulativo potencial para varios niveles de disponibilidad.

| % de tiempo de actividad | Tiempo de inactividad por semana | Tiempo de inactividad por mes | Tiempo de inactividad por año |
|----------|-------------------|--------------------|-------------------|
| 99% | 1,68 horas | 7,2 horas | 3,65 días |
| 99,9 % | 10 minutos | 43,2 minutos | 8,76 horas |
| 99,95 % | 5 minutos | 21,6 minutos | 4,38 horas |
| 99,99% | 1 minuto | 4,32 minutos | 52,56 minutos |
| 99,999 % | 6 segundos | 26 segundos | 5,26 minutos |

Tenga en cuenta que un tiempo de actividad del 99 % podría traducirse en una interrupción del servicio de casi 2 horas por semana. Para muchas aplicaciones, especialmente aquellas orientadas al consumidor, no es un objetivo de nivel de servicio aceptable. Por otro lado, cinco 9 (99,999 %) significa que no hay más de 5 minutos de tiempo de inactividad en un *año*. Resulta difícil simplemente detectar un problema con tanta rapidez, por no hablar de solucionarlo. Para obtener una disponibilidad muy alta (99,99 % o superior), la recuperación de un error no puede depender de la intervención manual. La aplicación debe diagnosticarse y recuperarse por sí misma, y es en ese punto en el que la resistencia se convierte en fundamental.

En Azure, el Acuerdo de Nivel de Servicio (SLA) explica los compromisos de Microsoft en cuanto a tiempo de actividad y conectividad. Si el Acuerdo de Nivel de Servicio para un servicio determinado es del 99,95 %, significa que debe esperar que el servicio esté disponible un 99,95 % del tiempo.

Las aplicaciones a menudo dependen de varios servicios. En general, la probabilidad de que cualquier servicio tenga un tiempo de inactividad es independiente. Por ejemplo, imagine que la aplicación depende de dos servicios, cada uno con un SLA del 99,9 %. El SLA compuesto para ambos servicios es del 99,9 % &times; 99,9 % &asymp; 99,8 %, o ligeramente menor que cada servicio por sí mismo. 

Use la [lista de comprobación de disponibilidad][availability-checklist] para revisar el diseño desde un punto de vista de la disponibilidad.

### <a name="availability-guidance"></a>Guía de disponibilidad

- [Design patterns for availability][availability-patterns] (Patrones de diseño para disponibilidad)
- Procedimientos recomendados: [Autoscaling][autoscale] (Escalado automático), [Background jobs][background-jobs] (Trabajos en segundo plano)

## <a name="resiliency"></a>Resistencia

La resistencia es la capacidad de un sistema de recuperarse de los errores y seguir funcionando. El objetivo de la resistencia es devolver la aplicación a un estado plenamente operativo después de un error. La resistencia está estrechamente relacionada con la disponibilidad.

En el desarrollo de aplicaciones tradicional se ha hecho especial hincapié en la reducción del tiempo medio entre errores. Se invertían esfuerzos en intentar impedir que se produjeran errores del sistema. En la informática en la nube se requiere una mentalidad distinta como consecuencia de diversos factores:

- Los sistemas distribuidos son complejos, y un error en un punto puede potencialmente reproducirse en todo el sistema.
- Los costos de los entornos de nube se mantienen bajos gracias al uso de hardware básico, por lo que deben esperarse errores de hardware ocasionales. 
- Las aplicaciones a menudo dependen de servicios externos, que podrían sufrir tiempos de inactividad o limitar a los usuarios de gran volumen. 
- Los usuarios actuales esperan una aplicación que esté disponible las 24 horas del día, los 7 días de la semana, sin quedarse nunca sin conexión.

Todos estos factores implican que las aplicaciones en la nube deben diseñarse para esperar errores ocasionales y recuperarse de ellos. Azure cuenta con muchas características de resistencia ya integradas en la plataforma. Por ejemplo, 

- Azure Storage, SQL Database y Cosmos DB proporcionan replicación de datos integrada, tanto dentro de una región como entre regiones.
- Las instancias de Azure Managed Disks se colocan automáticamente en unidades de escalado de almacenamiento diferentes a fin de limitar los efectos de los errores de hardware.
- Las máquinas virtuales en un conjunto de disponibilidad están repartidas entre varios dominios de error. Un dominio de error es un grupo de máquinas virtuales que comparten una fuente de alimentación común y un conmutador de red. El hecho de repartir las máquinas virtuales entre dominios de error limita el impacto de errores de hardware físico, interrupciones de red o cortes de alimentación eléctrica.

Dicho esto, aún necesita trabajar en la resistencia de su aplicación. Las estrategias de resistencia pueden aplicarse a todos los niveles de la arquitectura. Algunas de las mitigaciones son de naturaleza más táctica; por ejemplo, volver a intentar una llamada remota después de un error transitorio de la red. Otras mitigaciones son más estratégicas, como la conmutación por error de toda una aplicación en una región secundaria. Las mitigaciones tácticas pueden hacer una gran diferencia. Aunque es poco frecuente que toda una región experimente una interrupción, los problemas transitorios, como la congestión de la red, son más frecuentes, así que céntrese en estos primero. También es importante contar con una supervisión y un diagnóstico adecuados para detectar los errores en el momento en que se producen, así como para encontrar las causas principales.

Cuando se diseña una aplicación para que sea resistente, se deben comprender los requisitos de disponibilidad. ¿Cuánto tiempo de inactividad es aceptable? Depende en parte del costo. ¿Cuánto le costará el tiempo de inactividad potencial a su negocio? ¿Cuánto debe invertir para que la aplicación tenga alta disponibilidad?

Use la [lista de comprobación de resistencia][resiliency-checklist] para revisar el diseño desde un punto de vista de la resistencia.

### <a name="resiliency-guidance"></a>Guía de resistencia

- [Diseño de aplicaciones resistentes de Azure][resiliency]
- [Design patterns for resiliency][resiliency-patterns] (Patrones de diseño para la resistencia)
- Prácticas recomendadas: [Transient fault handling ][transient-fault-handling] (Control de errores transitorios), [Retry guidance for specific services][retry-service-specific] (Orientaciones de reintento para servicios específicos)

## <a name="management-and-devops"></a>DevOps y administración

Este fundamento abarca los procesos de las operaciones que mantienen a una aplicación ejecutándose en producción.

Las implementaciones deben ser confiables y predecibles. Se deben automatizar para reducir la posibilidad de que ocurran errores humanos. Deben ser un proceso rápido y rutinario, de manera que no ralenticen la publicación de nuevas características o correcciones de errores. Igualmente importante, debe ser capaz de revertir o poner al día la aplicación rápidamente en caso de que tenga problemas.

La supervisión y el diagnóstico son fundamentales. Las aplicaciones en la nube se ejecutan en un centro de datos remoto en el que no tiene un control completo de la infraestructura ni, en algunos casos, del sistema operativo. En una aplicación de gran tamaño, no resulta práctico volver a iniciar sesión en las máquinas virtuales para solucionar un problema o para examinar archivos de registro. Con los servicios de PaaS, puede incluso que no exista una máquina virtual dedicada en la que iniciar sesión. La supervisión y el diagnóstico proporcionan una visión general del sistema, para que sepa cuándo y dónde se producen errores. Todos los sistemas deben ser observables. Utilice un esquema de registro común y coherente que le permita correlacionar eventos en todos los sistemas.

El proceso de supervisión y diagnóstico consta de varias fases distintas:

- Instrumentación. Generar datos sin procesar, desde registros de aplicaciones, registros de servidor web o diagnósticos integrados en la plataforma de Azure, y otras fuentes.
- Recopilación y almacenamiento. Consolidar los datos en un solo lugar.
- Análisis y diagnóstico. Para solucionar problemas y ver el estado general.
- Visualización y alertas. Usar datos de telemetría para detectar tendencias o alertar al equipo de operaciones.

Use la [lista de comprobación de DevOps][devops-checklist] para revisar el diseño de un punto de vista de DevOps y administración.

### <a name="management-and-devops-guidance"></a>Guía de DevOps y administración

- [Design patterns for management and monitoring][management-patterns] (Patrones de diseño para la administración y supervisión)
- Procedimientos recomendados: [Monitoring and diagnostics][monitoring] (Supervisión y diagnóstico).

## <a name="security"></a>Seguridad

Debe pensar en la seguridad a lo largo de todo el ciclo de vida de una aplicación, desde el diseño y la implementación a la aplicación y las operaciones. La plataforma Azure proporciona protección contra diversas amenazas, como la intrusión de red y los ataques de DDoS. Sin embargo, aún tendrá que trabajar en la seguridad de la aplicación y los procesos de DevOps.

Estas son algunas áreas de seguridad generales que debe tener en cuenta. 

### <a name="identity-management"></a>Administración de identidades

Considere el uso de Azure Active Directory (Azure AD) para autenticar y autorizar a los usuarios. Azure AD es un servicio de administración de identidades y acceso completamente administrado. Puede usarlo para crear dominios que existen exclusivamente en Azure o integrarlo con las identidades de Active Directory locales. Azure AD también se integra con Office 365, Dynamics CRM Online y muchas aplicaciones de SaaS de terceros. En el caso de las aplicaciones orientadas al consumidor, Azure Active Directory B2C permite a los usuarios autenticarse con sus cuentas sociales existentes (como Facebook, Google o LinkedIn) o crear una nueva cuenta de usuario que se administra mediante Azure AD.

Si desea integrar un entorno de Active Directory local con una red de Azure, tiene a su disposición varios enfoques en función de los requisitos. Para obtener más información, consulte nuestras arquitecturas de referencia de [administración de identidades][identity-ref-arch].

### <a name="protecting-your-infrastructure"></a>Protección de su infraestructura 

Controle el acceso a los recursos de Azure que implemente. Cada suscripción de Azure tiene una [relación de confianza][ad-subscriptions] con un inquilino de Azure AD. Use [el control de acceso basado en rol][rbac] (RBAC) para conceder los permisos adecuados a los recursos de Azure a los usuarios de su organización. Conceda el acceso mediante la asignación de un rol de RBAC a usuarios o grupos en un ámbito determinado. El ámbito puede ser una suscripción, un grupo de recursos o incluso un solo recurso. [Audite][resource-manager-auditing] todos los cambios de infraestructura. 

### <a name="application-security"></a>Seguridad de las aplicaciones

En general, los procedimientos recomendados de seguridad para el desarrollo de aplicaciones todavía se aplican en la nube. Estos incluyen elementos como el uso de SSL en todas partes, la protección contra ataques CSRF y XSS, la prevención de ataques por inyección de código SQL, etc. 

Las aplicaciones en la nube suelen utilizar servicios administrados que tienen claves de acceso. No las inserte nunca en el control de código fuente. Considere la posibilidad de almacenar los secretos de la aplicación en Azure Key Vault.

### <a name="data-sovereignty-and-encryption"></a>Cifrado y soberanía de datos

Asegúrese de que los datos permanecen en la zona geopolítica correcta al utilizar la alta disponibilidad de Azure. El almacenamiento con replicación geográfica de Azure usa el concepto de [región emparejada][paired-region] en la misma región geopolítica. 

Use Key Vault para proteger los secretos y las claves criptográficas. Al usar Key Vault, podrá cifrar claves y secretos mediante el uso de claves que están protegidas por los módulos de seguridad de hardware (HSM). Muchos servicios de base de datos y almacenamiento de Azure admiten el cifrado de datos en reposo; por ejemplo, [Azure Storage][storage-encryption], [Azure SQL Database][sql-db-encryption], [Azure SQL Data Warehouse][data-warehouse-encryption] y [Cosmos DB][cosmosdb-encryption].

### <a name="security-resources"></a>Recursos de seguridad

- [Azure Security Center][security-center] proporciona una supervisión de la seguridad y una administración de directivas integradas en suscripciones de Azure. 
- [Documentación de Azure Security][security-documentation]
- [Centro de confianza de Microsoft][trust-center]



<!-- links -->

[dr-guidance]: ../resiliency/disaster-recovery-azure-applications.md
[identity-ref-arch]: ../reference-architectures/identity/index.md
[resiliency]: ../resiliency/index.md

[ad-subscriptions]: /azure/active-directory/active-directory-how-subscriptions-associated-directory
[data-warehouse-encryption]: /azure/data-lake-store/data-lake-store-security-overview#data-protection
[cosmosdb-encryption]: /azure/cosmos-db/database-security
[rbac]: /azure/active-directory/role-based-access-control-what-is
[paired-region]: /azure/best-practices-availability-paired-regions
[resource-manager-auditing]: /azure/azure-resource-manager/resource-group-audit
[security-blog]: https://azure.microsoft.com/blog/tag/security/
[security-center]: https://azure.microsoft.com/services/security-center/
[security-documentation]: /azure/security/
[sql-db-encryption]: /azure/sql-database/sql-database-always-encrypted-azure-key-vault
[storage-encryption]: /azure/storage/storage-service-encryption
[trust-center]: https://azure.microsoft.com/support/trust-center/
 

<!-- patterns -->
[availability-patterns]: ../patterns/category/availability.md
[management-patterns]: ../patterns/category/management-monitoring.md
[resiliency-patterns]: ../patterns/category/resiliency.md
[scalability-patterns]: ../patterns/category/performance-scalability.md


<!-- practices -->
[autoscale]: ../best-practices/auto-scaling.md
[background-jobs]: ../best-practices/background-jobs.md
[caching]: ../best-practices/caching.md
[cdn]: ../best-practices/cdn.md
[data-partitioning]: ../best-practices/data-partitioning.md
[monitoring]: ../best-practices/monitoring.md
[retry-service-specific]: ../best-practices/retry-service-specific.md
[transient-fault-handling]: ../best-practices/transient-faults.md


<!-- checklist -->
[availability-checklist]: ../checklist/availability.md
[devops-checklist]: ../checklist/dev-ops.md
[resiliency-checklist]: ../checklist/resiliency.md
[scalability-checklist]: ../checklist/scalability.md
