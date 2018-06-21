---
title: Lista de comprobación de DevOps
description: Lista de comprobación que proporciona guía relacionada con DevOps.
author: dragon119
ms.date: 01/10/2018
ms.custom: checklist
ms.openlocfilehash: 2e338d2f2e61b404223001a61f44e06e89e7f563
ms.sourcegitcommit: e67b751f230792bba917754d67789a20810dc76b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/06/2018
ms.locfileid: "30847066"
---
# <a name="devops-checklist"></a>Lista de comprobación de DevOps

DevOps es la integración de desarrollo, control de calidad y operaciones de TI en una referencia cultural unificada y un conjunto de procesos para la entrega de software. Utilice esta lista de comprobación como un punto de partida para evaluar su referencia cultural y proceso de DevOps.

## <a name="culture"></a>Referencia cultural

**Asegúrese de la alineación empresarial con el negocio entre organizaciones y equipos.** Los conflictos de recursos, propósito, objetivos y prioridades en una organización pueden suponer un riesgo para las operaciones correctas. Asegúrese de que los equipos de negocio, desarrollo y operaciones están alineados.

**Asegúrese de que todo el equipo conoce el ciclo de vida del software.** El equipo debe conocer el ciclo de vida general de la aplicación y la parte del ciclo de vida de la aplicación en que se encuentra la aplicación. Esto ayuda a todos los miembros del equipo a saber lo que deberían estar haciendo y lo que deberían planear y preparar.

**Reduzca el tiempo de ciclo.** El objetivo es minimizar el tiempo que se tarda en pasar de las ideas a un software desarrollado que se pueda usar. Limite el tamaño y ámbito de las versiones individuales para que no crezca la carga de prueba. Automatice los procesos de compilación, prueba, configuración e implementación siempre que sea posible. Elimine todos los obstáculos para la comunicación entre los desarrolladores, así como entre los desarrolladores y las operaciones. 

**Revise y mejore los procesos.** Sus procesos y procedimientos, tanto automatizados como manuales, nunca son finales. Configure revisiones periódicas de los flujos de trabajo actuales, procedimientos y documentación, con el objetivo de mejorar constantemente.

**Realice un planeamiento proactivo.** Planee de manera proactiva por si aparecen errores. Tenga listos los procesos para identificar rápidamente los problemas cuando se produzcan, escalarlos a los miembros correctos del equipo para solucionarlos y confirmar la resolución.

**Aprenda de los errores.** Los errores son inevitables, así que lo importante es aprender de ellos para no repetirlos. Si se produce un error de funcionamiento, evalúe las prioridades el problema, documente la causa y la solución, comparta todo lo que haya aprendido. Siempre que sea posible, actualice los procesos de compilación para detectar automáticamente ese tipo de error en el futuro.

**Optimize la velocidad de los datos y su recopilación.** Toda mejora planeada es una hipótesis. Trabaje en los menos incrementos posibles. Trate las ideas nuevas como experimentos. Instrumente los experimentos para que pueda recopilar datos de producción para evaluar su eficacia. Esté preparado para que se produzca un fracaso y responder rápido a los errores si la hipótesis es incorrecta.

**Deje tiempo para el aprendizaje.** Tanto los errores como los aciertos proporcionan buenas oportunidades para el aprendizaje. Antes de pasar a nuevos proyectos, deje tiempo suficiente para recopilar las lecciones importantes y asegúrese de que el equipo las absorbe. Dé también tiempo al equipo para desarrollar habilidades, experimentar y obtener información acerca de las nuevas herramientas y técnicas. 

**Documente las operaciones.** Documente todas las herramientas, procesos y tareas automatizadas con el mismo nivel de calidad que el código de producto. Documente el diseño actual y la arquitectura de los sistemas que admite, junto con los procesos de recuperación y otros procedimientos de mantenimiento. Céntrese en los pasos que realmente realiza, no en procesos que, teóricamente, son óptimos. Revise y actualice la documentación periódicamente. En el caso del código, asegúrese de que se incluyen comentarios significativos, sobre todo en las API públicas, y use herramientas que generen automáticamente documentación del código siempre que sea posible. 

**Comparta sus conocimientos.** La documentación sólo es útil si se sabe que existe y que se puede encontrar. Asegúrese de que la documentación está organizada y que se puede descubrir fácilmente. Sea creativo: use bolsas marrones (presentaciones informales), vídeos o boletines para compartir sus conocimientos.

## <a name="development"></a>Desarrollo

**Proporcione a los programadores entornos similares al de producción.** Si los entornos de desarrollo y prueba son distintos del de producción, es difícil probar y diagnosticar problemas. Por consiguiente, los entornos de desarrollo y prueba deben parecerse tanto como sea posible al de producción. Asegúrese de que los datos de prueba sean coherentes con los datos que se usan en producción, aunque sean datos de ejemplo, no datos de producción real (por motivos de privacidad o de cumplimiento). Planee la generación y anonimización de datos de prueba de ejemplo.

**Asegúrese de que todos los miembros del equipo autorizados pueden aprovisionar la infraestructura e implementar la aplicación.** La configuración de recursos similares a los de producción y la implementación de la aplicación no deberían implicar tareas manuales complicadas ni debería ser necesario tener conocimientos técnicos del sistema. Cualquier usuario con los permisos adecuados debería poder crear o implementar recursos similares a los de producción sin tener que recurrir al equipo de operaciones. 

> Esta recomendación no implica que cualquier usuario pueda insertar actualizaciones directas en la implementación de producción. Más bien se refiere a reducir la fricción para que los equipos de desarrollo y control de calidad creen entornos similares a los de producción.

**Instrumente la aplicación para obtener información.** Para conocer el estado de su aplicación, debe saber qué tal funciona y si han aparecido problemas o errores. Incluya siempre la instrumentación como un requisito de diseño y genérela en la aplicación desde el principio. La instrumentación debe incluir el registro de eventos para el análisis de la causa principal, pero también datos de telemetría y métrica para supervisar tanto el estado general de la aplicación como su uso.

**Realice un seguimiento de su deuda técnica.** En muchos proyectos, hasta cierto punto, las programaciones del lanzamiento pueden tener prioridad sobre la calidad del código. Si esto ocurre, realice siempre un seguimiento. Documente todos los accesos directos u otras implementaciones que no sean óptimas y programe la hora en el futuro para volver sobre dichos problemas.

**Considere la posibilidad de insertar las actualizaciones directamente en producción.** Para reducir el tiempo global del ciclo de lanzamiento, considere la posibilidad de insertar en producción confirmaciones de código probadas correctamente. Use la [activación/desactivación de funcionalidades][feature-toggles] para controlar qué características están habilitadas. Esto le permite usar la activación o desactivación de funcionalidades para pasar rápidamente del desarrollo al lanzamiento. La activación/desactivación también es útil cuando se realizan pruebas, como los [lanzamientos controlados][canary-release], donde una característica determinada se implementa en un subconjunto del entorno de producción.

## <a name="testing"></a>Prueba

**Automatice las pruebas.** La realización de pruebas manuales del software es una tarea tediosa y propensa a errores. Automatice las tareas comunes de las pruebas e integre estas últimas en los procesos de compilación. Las pruebas automatizadas garantizan una reproducibilidad y cobertura coherentes de las pruebas. Las pruebas de la interfaz de usuario integradas también las debe realizar una herramienta automatizada. Azure ofrece recursos de prueba y desarrollo que pueden servir de ayuda a la hora de configurar y ejecutan las pruebas. Para más información, consulte [Desarrollo y pruebas][dev-test].

**Realice pruebas para detectar errores.** Si un sistema no se puede conectar a un servicio, ¿cómo responde? ¿Se puede recuperar cuando el servicio vuelva a estar disponible? Haga que las pruebas de la inserción de errores sean una parte estándar de la revisión en los entornos de prueba y de ensayo. Cuando el proceso y las prácticas de las pruebas estén maduros, considere la posibilidad de ejecutar dichas pruebas en producción. 

**Realice pruebas en producción.** El proceso de lanzamiento no termina con el paso de la implementación a producción. Tenga listas las pruebas necesarias para asegurarse de que el código implementado funciona según lo previsto. En el caso de las implementaciones que se actualizan con poca frecuencia, programe las pruebas en producción como parte habitual del mantenimiento.

**Automatice las pruebas de rendimiento para identificar rápidamente los problemas de rendimiento.** El impacto de un problema de rendimiento serio puede tan grave como un error en el código. Aunque las pruebas automáticas de funcionalidad pueden evitar errores en la aplicación, es posible que no detecten problemas de rendimiento. Defina los objetivos de rendimiento aceptables para métricas como la latencia, los tiempos de carga y el uso de recursos. Incluya pruebas de rendimiento automatizadas en la canalización de versión para asegurarse de que la aplicación cumple los objetivos.

**Realice pruebas de capacidad.** Una aplicación podría funcionan correctamente durante las pruebas y qué, después, aparezcan problemas en producción debido a limitaciones de escala o recurso. Defina siempre los límites de uso y capacidad máxima esperada. Realice una prueba para asegurarse de que la aplicación puede controlar dichos límites, pero pruebe también lo que ocurre cuando se superan. Las pruebas de capacidad deben realizarse a intervalos regulares.

Después de la versión inicial, hay que ejecutar las pruebas de capacidad y rendimiento cada vez que se realicen actualizaciones en el código de producción. Use datos históricos para ajustar las pruebas y determinar qué tipos de pruebas deben realizarse.

**Realice pruebas de seguridad automatizadas.** Asegurarse de que su aplicación es segura es tan importante como probar cualquier otra funcionalidad. Haga que las pruebas de seguridad automatizadas sean una parte estándar del proceso de compilación e implementación. Programe pruebas de seguridad y exámenes de vulnerabilidades periódicos en las aplicaciones, en los que se supervise si hay puertos abiertos, ataques y puntos de conexión. Las realización automática de pruebas no elimina la necesidad de realizar revisiones de seguridad en profundidad a intervalos regulares.

**Realice pruebas automatizadas de continuidad empresarial.** Desarrolle pruebas de continuidad empresarial a gran escala, lo que incluye la conmutación por error y la recuperación de copias de seguridad. Configure procesos automatizados para realizar estas pruebas de forma periódica.

## <a name="release"></a>Release

**Automatice las implementaciones.** Automatice la implementación de la aplicación en los entornos de prueba, almacenamiento provisional y producción. La automatización permite que las implementaciones sean más rápidas y confiables, y garantiza que sean coherentes en todos los entornos compatibles. Elimina el riesgo de error humano que conllevan las implementaciones manuales. También facilita que los lanzamientos se programen para los momentos más apropiados, lo que reduce los efectos de un posible tiempo de inactividad.

**Use la integración continua.** La integración continua (CI) es la práctica de combinar todo el código de desarrollo en un código base según una programación periódica y, después, realizar automáticamente procesos de compilación y prueba estándar. La integración continua garantiza que todo un equipo puede trabajar simultáneamente en un código base sin que se produzcan conflictos. También se garantiza que los defectos que haya en el código se encuentran lo antes posible. Si es posible, el proceso de integración continua se debe ejecutar cada vez que un código se confirma o se inserta en el repositorio. Pero como mínimo se debe ejecutar una vez al día.

> Considere la posibilidad de usar un [modelo de desarrollo basado en un tronco][trunk-based]. En este modelo, los desarrolladores confirman en una única rama (el tronco). Hay un requisito que confirma que nunca interrumpe la compilación. Este modelo facilita la integración continua, ya que todo el trabajo de las funciones se realiza en el tronco y los conflictos de fusión mediante combinación se resuelven cuando se produce la confirmación.

**Considere la posibilidad de usar la entrega continua.** La entrega continua (CD) es la práctica consistente en garantizar que el código está siempre listo para implementarse, mediante la compilación, prueba e implementación automáticas del código en entornos similares al de producción. La incorporación de la entrega continua para crear una canalización de CI/CD completa le ayudará a detectar defectos en el código defectos lo antes posible y garantiza de que las actualizaciones probadas correctamente se pueden lanzar en muy poco tiempo.

> La *implementación* continua es un proceso adicional que implementa en producción todas las actualizaciones que han pasado por la canalización de CI/CD. La implementación continua requiere unas sólidas pruebas automáticas y un planeamiento avanzado del proceso, y puede que no sea adecuada para todos los equipos.

**Realice pequeños cambios incrementales.** Los grandes cambios en el código tienen un mayor potencial de introducir errores. Siempre que sea posible, realice cambios pequeños. Así se limitan los posibles efectos de cada cambio y se facilita la comprensión y depuración de los problemas.

**Controle la exposición a los cambios.** Asegúrese de que controla en qué momento ven las actualizaciones los usuarios finales. Considere la posibilidad de usar la activación/desactivación de funcionalidades para controlar en qué momento se habilitan las funcionalidades para los usuarios finales.

**Implemente estrategias de administración de versiones para reducir el riesgo de implementación.** La implementación de una actualización de un aplicación en producción siempre conlleva cierto riesgo. Para minimizarlo, utilice estrategias como los [lanzamientos controlados][canary-release] o las [implementaciones azules-verdes][blue-green] para implementar las actualizaciones en un subconjunto de usuarios. Confirme que la actualización funciona según lo previsto y, después, implemente la actualización en el resto del sistema.

**Documente todos los cambios.** Tanto las actualizaciones secundarias como los cambios en la configuración pueden ser una fuente de confusión y de conflictos entre versiones. Mantenga siempre un registro claro de todos los cambios que se realicen, sea cual sea su tamaño. Registre todos los cambios, lo que incluye las revisiones aplicadas, los cambios en las directivas y los cambios de configuración (en estos registros no se deben incluir datos confidenciales. Por ejemplo, registre que se ha actualizado una credencial y quién ha realizado el cambio, pero no registre las credenciales actualizadas). Todo el equipo debe poder ver el registro de los cambios. 

**Automatice las implementaciones.** Automatice todas las implementaciones y tenga listos sistemas que detecten cualquier problema que surja en la implementación. Tenga un proceso de mitigación para conservar los datos y el código existentes en producción, antes de que la actualización los reemplace en todas las instancias de producción. Tenga una forma automatizada de poner al día las correcciones o revertir los cambios.

**Considere la posibilidad de hacer que la infraestructura sea inmutable.** La infraestructura inmutable es el principio que indica que no se debe modificar la infraestructura una vez que se implementa en producción. Esto se debe a que, si lo hace, puede entrar en un estado en el que se han aplicado cambios ad hoc, lo que hace más difícil saber exactamente qué es lo que ha cambiado. El funcionamiento de la infraestructura inmutable se basa en el reemplazo de servidores enteros cuando se hace cualquier implementación nueva. Esto permite que tanto el código como el entorno se puedan probar e implementar como un bloque. Una vez implementados, los componentes de la infraestructura no se modifican hasta el siguiente ciclo de compilación e implementación. 

## <a name="monitoring"></a>Supervisión

**Asegúrese de los sistemas se pueden ver.** El equipo de operaciones siempre debe tener una visión clara del estado y mantenimiento de un sistema o servicio. Configure los puntos de conexión de mantenimiento externos para supervisar el estado y asegúrese de que las aplicaciones están codificadas para instrumentar las métricas de las operaciones. Utilice un esquema de común y consistente que le permita correlacionar eventos en todos los sistemas. [Azure Diagnostics][azure-diagnostics] y [Application Insights][app-insights] son el método estándar de seguimiento del estado y mantenimiento de los recursos de Azure. Microsoft [Operation Management Suite][oms] también proporciona una supervisión y administración centralizada de las soluciones híbridas o en la nube.

**Agregue y correlacione registros y métricas**. Un sistema de telemetría correctamente instrumentado proporcionará gran cantidad de datos de rendimiento sin procesar y de registros de eventos. Asegúrese de que tanto los datos de telemetría como los de registro se procesan y correlacionan rápidamente, con el fin de que el personal de operaciones siempre tenga una representación actualizada del mantenimiento del sistema. Organice y presente los datos de una forma que proporcione una visión cohesionada de los problemas, de forma que, siempre que sea posible, quede claro si los eventos están relacionados entre sí.

> Consulte en la directiva de retención corporativa los requisitos relativos a la forma en que se procesan los datos y al tiempo durante el que deben almacenarse. 

**Implemente alertas y notificaciones automatizadas.** Configurar herramientas de supervisión, como [Azure Monitor][azure-monitor] para que detecten patrones o condiciones que indique que hay o puede haber problemas, y envíe alertas a los miembros del equipo que pueden resolverlos. Ajuste las alertas para evitar falsos positivos.

**Supervise las fechas de expiración de los recursos.** Algunos recursos, como los certificados, expiran después de un período determinado. Asegúrese de realizar un seguimiento de los recursos que expiran, cuándo lo hacen y qué servicios o funciones dependen de ellos. Utilice procesos automatizados para supervisar estos recursos. Envíe una notificación al equipo de operaciones antes de que expire un activo y escálela si la expiración amenaza con interrumpir la aplicación.

## <a name="management"></a>Administración

**Automatice las tareas de las operaciones.** El control manual de los procesos de las operaciones repetitivas es algo propenso a errores. Automatice estas tareas siempre que sea posible para garantizar que la calidad y la ejecución sean coherentes. El código que implementa la automatización debe tener versiones en el control de código fuente. Al igual que con cualquier otro código, las herramientas de automatización se deben probar.

**Adopte un enfoque de infraestructura como código para realizar el aprovisionamiento.** Minimice la cantidad de configuración manual necesaria para aprovisionar los recursos. En su lugar, use scripts y las plantillas de [Azure Resource Manager][resource-manager]. Guarde los scripts y las plantillas en el control de código fuente, como haría con cualquier otro código que mantenga. 

**Considere el uso de contenedores.** Los contenedores proporcionan una interfaz estándar basado en paquetes para implementar las aplicaciones. Mediante el uso de contenedores, una aplicación se implementa mediante paquetes independientes que incluyen el software, las dependencias y los archivos necesarios para ejecutar la aplicación, lo que simplifica considerablemente el proceso de implementación. 

Los contenedores también crean una capa de abstracción entre la aplicación y el sistema operativo subyacente, lo que proporciona coherencia entre los entornos. Esta abstracción también puede aislar un contenedor de otros procesos o aplicaciones que se ejecutan en un host. 

**Implemente la resistencia y la recuperación automática.** La resistencia es la capacidad que tiene una aplicación para recuperarse de los errores. Las estrategias para lograr resistencia incluyen la realización de reintentos cuando los errores son transitorios y la conmutación por error a una instancia secundaria, o incluso a otra región. Para más información, consulte [Diseño de aplicaciones resistentes de Azure][resiliency]. Instrumente las aplicaciones para que los problemas se notifiquen de inmediato y pueda las interrupciones u otros errores del sistema.

**Tenga un manual de operaciones.** Un manual de operaciones o *runbook* documenta los procedimientos y la información de administración necesarios para que el personal de operaciones mantenga un sistema. Documente también tanto los escenarios de las operaciones como los planes de mitigación que podrían entran en juego en caso de error o cualquier otra alteración del servicio. Cree esta documentación durante el proceso de desarrollo y manténgala actualizada posteriormente. Esto es un documento vivo y se debe revisar, probar y mejorar de forma periódica. 

La documentación compartida es crítica. Anime a los miembros del equipo a contribuir y a compartir sus conocimientos. Todo el equipo debe tener acceso a los documentos. Facilite a los miembros del equipo la tarea de mantener los documentos actualizados.

**Documente los procedimientos de guardia.** Asegúrese de que los procedimientos, las programaciones y las tareas de guardia se documentan y comparten con todos los miembros del equipo. Mantenga esta información actualizada en todo momento.

**Documente los procedimientos de escalado para las dependencias de terceros.** Si una aplicación depende de servicios de terceros externos que no controla directamente, debe tener un plan para tratar las interrupciones. Cree documentación para los procesos de mitigación planeados. Incluya los contactos de soporte técnico y las rutas de escalado.

**Use la administración de la configuración.** Los cambios en la configuración deben planearse, registrarse y las aplicaciones deben poder verlos. Todo esto puede hacerse mediante una base de datos de administración de la configuración o un enfoque de configuración como código. La configuración se debe auditar periódicamente, con el fin de asegurarse todo está como cabría esperar.

**Obtenga un plan de soporte técnico de Azure y conozca el proceso.** Azure ofrece una serie de [planes de soporte técnico][azure-support-plans]. Determine el plan que más se ajuste a sus necesidades y asegúrese de que todo el equipo sabe cómo utilizarlo. Los miembros del equipo deben conocer los detalles del plan, cómo funciona el proceso de soporte técnico y cómo abrir una incidencia de soporte técnico en Azure. Si espera un evento a gran escala, el personal de soporte técnico de Azure puede ayudarle a aumentar los límites de sus servicios. Para más información, consulte [Preguntas más frecuentes de soporte técnico de Azure](https://azure.microsoft.com/support/faq/).

**Siga los principios de privilegios mínimos al conceder acceso a los recursos.** Administre con precaución el acceso a los recursos. El acceso se debe denegar de forma predeterminada, salvo que a un usuario explícitamente se le concede explícitamente acceso a un recurso. Conceda a los usuarios exclusivamente el acceso que necesitan para completar sus tareas. Realice un seguimiento de los permisos de los usuario y realice auditorías de seguridad periódicas.

<strong>Use el control de acceso basado en rol</strong> La asignación de cuentas de usuario y de acceso a los recursos no debe ser un proceso manual. Use el [control de acceso basado en rol][rbac] (RBAC) para la concesión de acceso en función de las identidades y grupos de [Azure Active Directory][azure-ad]. 

**Utilice un sistema de seguimiento de errores para realizar el seguimiento de los problemas.** Sin una buena forma de realizar el seguimiento de los problemas, es fácil pasar por alto elementos, duplicar el trabajo o crear problemas adicionales. No confíe en comunicación interpersonal informal para realizar un seguimiento del estado de los errores. Use un herramienta de seguimiento de errores para registrar información detallada acerca de los problemas, asignar los recursos necesarios para solucionarlos y proporcionar una pista de auditoría de su progreso y estado. 

**Administre todos los recursos en un sistema de administración de cambios.** Todos los aspectos del proceso de DevOps se deben incluir en un sistema de administración y control de versiones para que los cambios pueden controlarse y auditar fácilmente. Esto incluye el código, la infraestructura, la configuración, la documentación y los scripts. Trate todos estos tipos de recursos como código en el proceso de prueba/compilación/revisión. 

**Use listas de comprobación.** Cree listas de comprobación de las operaciones para asegurarse de que siguen los procesos. Es habitual perder algo en un manual grande pero el uso de una lista de comprobación puede llamar la atención de detalles que, sin ella, se podrían pasar por alto. Mantenga las listas de comprobación y busque constantemente distintas formas de automatizar tareas y simplificar procesos.

Para más información acerca de DevOps, consulte [¿Qué es DevOps?][what-is-devops] en el sitio de Visual Studio.

<!-- links -->

[app-insights]: /azure/application-insights/
[azure-ad]: https://azure.microsoft.com/services/active-directory/
[azure-diagnostics]: /azure/monitoring-and-diagnostics/azure-diagnostics
[azure-monitor]: /azure/monitoring-and-diagnostics/monitoring-overview
[azure-support-plans]: https://azure.microsoft.com/support/plans/
[blue-green]: https://martinfowler.com/bliki/BlueGreenDeployment.html
[canary-release]:https://martinfowler.com/bliki/CanaryRelease.html
[dev-test]: https://azure.microsoft.com/solutions/dev-test/
[feature-toggles]: https://www.martinfowler.com/articles/feature-toggles.html
[oms]: https://www.microsoft.com/cloud-platform/operations-management-suite
[rbac]: /azure/active-directory/role-based-access-control-what-is
[resiliency]: ../resiliency/index.md
[resource-manager]: /azure/azure-resource-manager/
[trunk-based]: https://trunkbaseddevelopment.com/
[what-is-devops]: https://www.visualstudio.com/learn/what-is-devops/
