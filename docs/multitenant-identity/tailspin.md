---
title: Acerca de la aplicación Tailspin Surveys
description: Información general acerca de la aplicación Tailspin Surveys
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: index
pnp.series.next: authenticate
ms.openlocfilehash: 028f7940d2e3cd7e8e629554f8af290ec5fdd184
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
ms.locfileid: "24540063"
---
# <a name="the-tailspin-scenario"></a>El escenario de Tailspin

[![GitHub](../_images/github.png) Código de ejemplo][sample application]

Tailspin es una compañía ficticia que está desarrollando una aplicación SaaS llamada Surveys. Esta aplicación permite a las organizaciones crear y publicar encuestas en línea.

* Una organización puede suscribirse a la aplicación.
* Después de que la organización se ha suscrito, los usuarios pueden iniciar sesión en la aplicación con las credenciales de la organización.
* Los usuarios pueden crear, editar y publicar encuestas.

> [!NOTE]
> Para empezar a trabajar con la aplicación, consulte [Running the Surveys application ] (Ejecución de la aplicación Surveys).
> 
> 

## <a name="users-can-create-edit-and-view-surveys"></a>Los usuarios pueden crear, editar y visualizar encuestas.
Un usuario no autenticado puede ver todas las encuestas que ha creado, o sobre las que tiene derechos de colaborador, y crear nuevas encuestas. Observe que el usuario ha iniciado sesión con su identidad organizativa, `bob@contoso.com`.

![Aplicación Surveys](./images/surveys-screenshot.png)

Esta captura de pantalla muestra la página Edit Survey (Editar encuesta):

![Editar encuesta](./images/edit-survey.png)

Los usuarios también pueden ver todas las encuestas creadas por otros usuarios del mismo inquilino.

![Encuestas de inquilinos](./images/tenant-surveys.png)

## <a name="survey-owners-can-invite-contributors"></a>Los propietarios de encuestas pueden invitar a colaboradores
Cuando un usuario crea una encuesta, puede invitar a otras personas a ser colaboradores en la misma. Los colaboradores pueden modificar la encuesta, pero no pueden eliminarla ni publicarla.  

![Agregar colaborador](./images/add-contributor.png)

Un usuario puede agregar colaboradores de otros inquilinos, lo que permite compartir recursos entre inquilinos. En esta captura de pantalla, Bob (`bob@contoso.com`) está agregando a Alice (`alice@fabrikam.com`) como colaboradora a una encuesta que Bob ha creado.

Cuando Alice inicia sesión, ve la encuesta que aparece en la lista de "Surveys I can contribute to" (Encuestas en las que puedo colaborar).

![Colaborador de la encuesta](./images/contributor.png)

Tenga en cuenta que Alice inicia sesión en su propio inquilino, no como un invitado del inquilino Contoso. Alice dispone de permisos de colaborador solo para esa encuesta pero no puede ver otras encuestas desde el inquilino Contoso.

## <a name="architecture"></a>Arquitectura
La aplicación Surveys consta de un front-end web y un back-end de API web. Ambos se implementan mediante [ASP.NET Core].

La aplicación web utiliza Azure Active Directory (Azure AD) para autenticar a los usuarios. La aplicación web también llama a Azure AD para obtener tokens de acceso de OAuth 2 para la API Web. Los tokens de acceso se almacenan en Azure Redis Cache. La memoria caché habilita a varias instancias para compartir la misma caché de token (por ejemplo, en una granja de servidores).

![Arquitectura](./images/architecture.png)

[**Siguiente**][authentication]

<!-- Links -->

[authentication]: authenticate.md

[Running the Surveys application ]: ./run-the-app.md
[ASP.NET Core]: /aspnet/core
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
