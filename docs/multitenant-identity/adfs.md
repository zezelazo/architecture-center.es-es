---
title: "Federación con un servicio AD FS de un cliente"
description: "Cómo federar con el servicio AD FS de un cliente en una aplicación multiinquilino"
author: MikeWasson
ms:date: 07/21/2017
pnp.series.title: Manage Identity in Multitenant Applications
pnp.series.prev: token-cache
pnp.series.next: client-assertion
ms.openlocfilehash: 08bf567085a940287de310f61b9f447d0ce5d5ec
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 02/23/2018
---
# <a name="federate-with-a-customers-ad-fs"></a>Federación con un servicio AD FS de un cliente

Este artículo describe cómo una aplicación SaaS multiinquilino puede admitir la autenticación mediante Servicios de federación de Active Directory (AD FS), para federarse con AD FS de un cliente.

## <a name="overview"></a>Información general
Azure Active Directory (Azure AD) facilita el inicio de sesión de los usuarios desde inquilinos de Azure AD, incluidos los clientes de Office 365 y Dynamics CRM Online. Pero, ¿qué sucede con los clientes que usan instalaciones locales de Active Directory en una intranet corporativa?

Es una opción es que estos clientes sincronicen su instalación local de AD con Azure AD mediante [Azure AD Connect]. Sin embargo, algunos clientes quizás no puedan usar este método debido a la directiva de TI corporativa u otras razones. En ese caso, otra opción es federar mediante los Servicios de federación de Active Directory (AD FS).

Para habilitar este escenario:

* El cliente debe tener una granja de servidores de ADFS con conexión a Internet.
* El proveedor de SaaS implementa su propia granja de servidores de AD FS.
* El cliente y el proveedor de SaaS deben configurar la [confianza de federación]. Se trata de un proceso manual.

Hay tres roles principales en la relación de confianza:

* El servicio AD FS del cliente es el [asociado de cuentas], responsable de autenticar los usuarios del directorio AD del cliente y de crear los tokens de seguridad con las notificaciones de usuario.
* El servicio AD FS del proveedor de SaaS es el [asociado de recursos], que confía en el asociado de cuentas y recibe las notificaciones de usuario.
* La aplicación se configura como un usuario de confianza (RP) en el servicio AD FS del proveedor de SaaS.
  
  ![confianza de federación](./images/federation-trust.png)

> [!NOTE]
> En este artículo se da por hecho que la aplicación usa OpenID Connect como protocolo de autenticación. Otra opción es usar WS-Federation.
> 
> Para OpenID Connect, el proveedor de SaaS debe usar AD FS 2016, que se ejecuta en Windows Server 2016. AD FS 3.0 no admite OpenID Connect.
> 
> ASP.NET Core no incluye compatibilidad integrada para WS-Federation.
> 
> 

Para obtener un ejemplo del uso de WS-Federation con ASP.NET 4, consulte el [ejemplo active-directory-dotnet-webapp-wsfederation][active-directory-dotnet-webapp-wsfederation].

## <a name="authentication-flow"></a>Flujo de autenticación
1. Cuando el usuario hace clic en "iniciar sesión", la aplicación se redirige a un punto de conexión de OpenID Connect en el servicio AD FS del proveedor de SaaS.
2. El usuario escribe su nombre de usuario de la organización ("`alice@corp.contoso.com`"). AD FS usa la detección del dominio de inicio para redirigir al servicio AD FS del cliente, donde el usuario escribe sus credenciales.
3. El servicio AD FS del cliente envían notificaciones de usuario al servicio AD FS del proveedor de SaaS, mediante WF-Federation (o SAML).
4. Las notificaciones se dirigen desde AD FS a la aplicación mediante OpenID Connect. Esto requiere una transición de protocolo desde WS-Federation.

## <a name="limitations"></a>Limitaciones
De forma predeterminada, la aplicación de usuario de confianza recibe solo un conjunto fijo de notificaciones disponibles en id_token, que se muestra en la tabla siguiente. Con AD FS 2016, puede personalizar el valor de id_token en escenarios de OpenID Connect. Para obtener más información, vea [Tokens de identificador personalizado en AD FS](/windows-server/identity/ad-fs/development/customize-id-token-ad-fs-2016).

| Notificación | DESCRIPCIÓN |
| --- | --- |
| aud |Audiencia. La aplicación para que la que se emitieron las notificaciones. |
| authenticationinstant |[Instante de autenticación]. La hora en que se produjo la autenticación. |
| c_hash |Valor de código hash. Se trata de un valor hash del contenido del token. |
| exp |[Fecha de expiración]. El momento después del cual el token ya no se aceptará. |
| iat |Emitido a las. La hora a la que se generó el token. |
| iss |Emisor. El valor de esta notificación siempre es el servicio AD FS del asociado de recursos. |
| Nombre |Nombre de usuario. Ejemplo: `john@corp.fabrikam.com` |
| nameidentifier |[Identificador de nombre]. El identificador para el nombre de la entidad para la que se emitió el token. |
| valor de seguridad |Nonce de sesión. Un valor único generado por AD FS para ayudar a evitar ataques de reproducción. |
| upn |Nombre principal de usuario (UPN). Ejemplo: `john@corp.fabrikam.com` |
| pwd_exp |Período de expiración de contraseña. El número de segundos hasta que caduca la contraseña del usuario o un secreto de autenticación similar, como un PIN. |

> [!NOTE]
> La notificación "iss" contiene el AD FS del asociado (esta notificación normalmente identificará al proveedor de SaaS como emisor). No identifica el AD FS del cliente. Puede encontrar el dominio del cliente como parte de UPN.
> 
> 

El resto de este artículo describe cómo configurar la relación de confianza entre el usuario de confianza (la aplicación) y el asociado de cuentas (el cliente).

## <a name="ad-fs-deployment"></a>Implementación de AD FS
El proveedor de SaaS puede implementar AD FS en máquinas virtuales de Azure o locales. Por motivos de seguridad y disponibilidad, son importantes las siguientes directrices:

* Implemente al menos dos servidores de AD FS y dos servidores proxy de AD FS para lograr la mejor disponibilidad del servicio AD FS.
* Los controladores de dominio y los servidores de AD FS nunca deben exponerse directamente a Internet y deben estar en una red virtual con acceso directo a ellos.
* Se deben usar servidores proxy de aplicación web (antes, servidores proxy de AD FS) para publicar servidores de AD FS en Internet.

Para configurar una topología similar en Azure, es necesario usar redes virtuales, NSG, máquinas virtuales de VM y conjuntos de disponibilidad. Para más información, consulte [Directrices para implementar Windows Server Active Directory en Azure Virtual Machines][active-directory-on-azure].

## <a name="configure-openid-connect-authentication-with-ad-fs"></a>Configuración de la autenticación de OpenID Connect con AD FS
El proveedor de SaaS debe habilitar OpenID Connect entre la aplicación y AD FS. Para ello, agregue un grupo de aplicaciones a AD FS.  Encontrará instrucciones detalladas en esta [entrada de blog], en “Setting up a Web App for OpenId Connect sign in AD FS” (Configuración de una aplicación web para el inicio de sesión de OpenId Connect en AD FS). 

Después, configure el middleware de OpenID Connect. El punto de conexión de metadatos es `https://domain/adfs/.well-known/openid-configuration`, donde domain es el dominio del servicio AD FS del proveedor de SaaS.

Normalmente se puede combinar con otros puntos de conexión de OpenID Connect (como AAD). Necesitará dos botones de inicio de sesión diferentes o alguna otra forma de distinguirlos, para enviar al usuario al punto de conexión de autenticación correcto.

## <a name="configure-the-ad-fs-resource-partner"></a>Configuración del asociado de recursos de AD FS
El proveedor de SaaS debe hacer lo siguiente para cada cliente que desee conectarse mediante AD FS:

1. Agregar confianza para el proveedor de notificaciones.
2. Agregar reglas de notificaciones.
3. Habilitar la detección del dominio principal.

Estos son los pasos con más detalle.

### <a name="add-the-claims-provider-trust"></a>Adición de confianza para el proveedor de notificaciones
1. En el Administrador del servidor, haga clic en **Herramientas** y, luego, seleccione **Administración de AD FS**.
2. En el árbol de consola, bajo **AD FS**, haga clic en **Confianzas de proveedores de notificaciones**. Seleccione **Agregar confianza de proveedor de notificaciones**.
3. Haga clic en **Iniciar** para iniciar el asistente.
4. Seleccione la opción "Importar datos acerca del proveedor de notificaciones publicado en línea o en una red local". Escriba el identificador URI del punto de conexión de metadatos de federación del cliente. (Ejemplo: `https://contoso.com/FederationMetadata/2007-06/FederationMetadata.xml`). Tendrá que obtenerlo del cliente.
5. Complete el asistente usando las opciones predeterminadas.

### <a name="edit-claims-rules"></a>Edición de reglas de notificaciones
1. Haga clic con el botón derecho en la confianza de proveedor de notificaciones recién agregada y seleccione **Editar reglas de notificaciones**.
2. Haga clic en **Agregar regla**.
3. Seleccione "Pasar a través o filtrar una notificación entrante" y haga clic en **Siguiente**.
   ![Asistente para agregar regla de notificación de transformación](./images/edit-claims-rule.png)
4. Escriba un nombre para la regla.
5. En "Tipo de notificación entrante", seleccione **UPN**.
6. Seleccione "Pasar todos los valores de notificación".
   ![Asistente para agregar regla de notificación de transformación](./images/edit-claims-rule2.png)
7. Haga clic en **Finalizar**
8. Repita los pasos 2 a 7 y especifique **Tipo de delimitador de notificación** para el tipo de notificación entrante.
9. Haga clic en **Aceptar** para completar el asistente.

### <a name="enable-home-realm-discovery"></a>Habilitación de la detección del dominio principal
Ejecute el siguiente script de PowerShell:

```
Set-ADFSClaimsProviderTrust -TargetName "name" -OrganizationalAccountSuffix @("suffix")
```

donde "name" es el nombre descriptivo de la confianza de proveedor de notificaciones, y "suffix" es el sufijo UPN del directorio AD del cliente (ejemplo, "corp.fabrikam.com").

Con esta configuración, los usuarios finales pueden escribir sus cuentas de la organización y AD FS selecciona automáticamente el proveedor de notificaciones correspondiente. Consulte [Personalizar las páginas de inicio de sesión de AD FS], en la sección "Configuración del proveedor de identidades para utilizar ciertos sufijos de correo electrónico".

## <a name="configure-the-ad-fs-account-partner"></a>Configuración del asociado de cuentas de AD FS
Los usuarios deben hacer lo siguiente:

1. Agregar una confianza para usuario de confianza.
2. Agregar reglas de notificaciones.

### <a name="add-the-rp-trust"></a>Adición de confianzas para usuarios de confianza
1. En el Administrador del servidor, haga clic en **Herramientas** y, luego, seleccione **Administración de AD FS**.
2. En el árbol de consola, bajo **AD FS**, haga clic en **Veracidades de usuarios de confianza**. Seleccione **Agregar confianza para usuario de confianza**.
3. Seleccione **Compatible con notificaciones** y haga clic en **Iniciar**.
4. En la página **Seleccionar origen de datos** , seleccione la opción "Importar datos acerca del proveedor de notificaciones publicado en línea o en una red local". Escriba el identificador URI del punto de conexión de metadatos de federación del proveedor de SaaS.
   ![Asistente para agregar relación de confianza para usuario autenticado](./images/add-rp-trust.png)
5. En la página **Especificar nombre para mostrar** , escriba cualquier nombre.
6. En la página **Elegir la directiva de control de acceso** , elija una directiva. Puede dar permiso a todos los usuarios de la organización o elegir un grupo de seguridad específico.
   ![Asistente para agregar relación de confianza para usuario autenticado](./images/add-rp-trust2.png)
7. Escriba los parámetros necesarios en el cuadro **Directiva**.
8. Haga clic en **Siguiente** para completar el asistente.

### <a name="add-claims-rules"></a>Adición de reglas de notificaciones
1. Haga clic con el botón derecho en la confianza para usuario de confianza recién agregada y seleccione **Editar directiva de emisión de notificación**.
2. Haga clic en **Agregar regla**.
3. Seleccione "Enviar atributos LDAP como notificaciones" y haga clic en **Siguiente**.
4. Escriba un nombre para la regla, por ejemplo, “UPN”.
5. En el **almacén de atributos**, seleccione **Active Directory**.
   ![Asistente para agregar regla de notificación de transformación](./images/add-claims-rules.png)
6. En la sección **Asignación de atributos LDAP** :
   * En **Atributo LDAP**, seleccione **User-Principal-Name**.
   * En **Tipo de notificación saliente**, seleccione **UPN**.
     ![Asistente para agregar regla de notificación de transformación](./images/add-claims-rules2.png)
7. Haga clic en **Finalizar**
8. Haga clic en **Agregar regla** de nuevo.
9. Seleccione “Enviar notificaciones con una regla personalizada” y haga clic en **No**.
10. Escriba un nombre para la regla, por ejemplo, "Delimitar tipo de notificación".
11. En **Regla personalizada**, escriba lo siguiente:
    
    ```
    EXISTS([Type == "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype"])=>
    issue (Type = "http://schemas.microsoft.com/ws/2014/01/identity/claims/anchorclaimtype",
          Value = "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/upn");
    ```
    
    Esta regla emite una notificación de tipo `anchorclaimtype`. La notificación indica al usuario de confianza que use el UPN como identificador inmutable del usuario.
12. Haga clic en **Finalizar**
13. Haga clic en **Aceptar** para completar el asistente.


<!-- Links -->
[Azure AD Connect]: /azure/active-directory/active-directory-aadconnect/
[confianza de federación]: https://technet.microsoft.com/library/cc770993(v=ws.11).aspx
[asociado de cuentas]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[asociado de recursos]: https://technet.microsoft.com/library/cc731141(v=ws.11).aspx
[Instante de autenticación]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.authenticationinstant%28v=vs.110%29.aspx
[Fecha de expiración]: http://tools.ietf.org/html/draft-ietf-oauth-json-web-token-25#section-4.1.
[Identificador de nombre]: https://msdn.microsoft.com/library/system.security.claims.claimtypes.nameidentifier(v=vs.110).aspx
[active-directory-on-azure]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[entrada de blog]: http://www.cloudidentity.com/blog/2015/08/21/OPENID-CONNECT-WEB-SIGN-ON-WITH-ADFS-IN-WINDOWS-SERVER-2016-TP3/
[Personalizar las páginas de inicio de sesión de AD FS]: https://technet.microsoft.com/library/dn280950.aspx
[sample application]: https://github.com/mspnp/multitenant-saas-guidance
[client assertion]: client-assertion.md
[active-directory-dotnet-webapp-wsfederation]: https://github.com/Azure-Samples/active-directory-dotnet-webapp-wsfederation
