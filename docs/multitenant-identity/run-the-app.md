---
title: "Ejecución de la aplicación Surveys"
description: "Cómo ejecutar localmente la aplicación de ejemplo Surveys"
author: MikeWasson
ms:date: 07/21/2017
ms.openlocfilehash: d17cd939c1172edd0947b30ea13657806060b5f1
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 11/14/2017
---
# <a name="run-the-surveys-application"></a><span data-ttu-id="9ba29-103">Ejecución de la aplicación Surveys</span><span class="sxs-lookup"><span data-stu-id="9ba29-103">Run the Surveys application</span></span>

<span data-ttu-id="9ba29-104">En este artículo se describe cómo ejecutar localmente la aplicación [Tailspin Surveys](./tailspin.md) desde Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="9ba29-104">This article describes how to run the [Tailspin Surveys](./tailspin.md) application locally, from Visual Studio.</span></span> <span data-ttu-id="9ba29-105">En estos pasos, no va a implementar la aplicación en Azure.</span><span class="sxs-lookup"><span data-stu-id="9ba29-105">In these steps, you won't deploy the application to Azure.</span></span> <span data-ttu-id="9ba29-106">Sin embargo, tendrá que crear algunos recursos de Azure &mdash; un directorio de Azure Active Directory (Azure AD) y una caché en Redis.</span><span class="sxs-lookup"><span data-stu-id="9ba29-106">However, you will need to create some Azure resources &mdash; an Azure Active Directory (Azure AD) directory and a Redis cache.</span></span>

<span data-ttu-id="9ba29-107">Este es un resumen de los pasos:</span><span class="sxs-lookup"><span data-stu-id="9ba29-107">Here is a summary of the steps:</span></span>

1. <span data-ttu-id="9ba29-108">Cree un directorio de Azure AD (inquilino) para la empresa ficticia Tailspin.</span><span class="sxs-lookup"><span data-stu-id="9ba29-108">Create an Azure AD directory (tenant) for the fictitious Tailspin company.</span></span>
2. <span data-ttu-id="9ba29-109">Registre la aplicación Surveys y la API web de back-end con Azure AD.</span><span class="sxs-lookup"><span data-stu-id="9ba29-109">Register the Surveys application and the backend web API with Azure AD.</span></span>
3. <span data-ttu-id="9ba29-110">Cree de una instancia de Azure Redis Cache.</span><span class="sxs-lookup"><span data-stu-id="9ba29-110">Create an Azure Redis Cache instance.</span></span>
4. <span data-ttu-id="9ba29-111">Configure la aplicación y cree una base de datos local.</span><span class="sxs-lookup"><span data-stu-id="9ba29-111">Configure application settings and create a local database.</span></span>
5. <span data-ttu-id="9ba29-112">Ejecute la aplicación y registre un nuevo inquilino.</span><span class="sxs-lookup"><span data-stu-id="9ba29-112">Run the application and sign up a new tenant.</span></span>
6. <span data-ttu-id="9ba29-113">Agregue roles de la aplicación a los usuarios.</span><span class="sxs-lookup"><span data-stu-id="9ba29-113">Add application roles to users.</span></span>

## <a name="prerequisites"></a><span data-ttu-id="9ba29-114">Requisitos previos</span><span class="sxs-lookup"><span data-stu-id="9ba29-114">Prerequisites</span></span>
-   <span data-ttu-id="9ba29-115">[Visual Studio 2017][VS2017]</span><span class="sxs-lookup"><span data-stu-id="9ba29-115">[Visual Studio 2017][VS2017]</span></span>
-   <span data-ttu-id="9ba29-116">Cuenta de [Microsoft Azure](https://azure.microsoft.com)</span><span class="sxs-lookup"><span data-stu-id="9ba29-116">[Microsoft Azure](https://azure.microsoft.com) account</span></span>

## <a name="create-the-tailspin-tenant"></a><span data-ttu-id="9ba29-117">Creación del inquilino de Tailspin</span><span class="sxs-lookup"><span data-stu-id="9ba29-117">Create the Tailspin tenant</span></span>

<span data-ttu-id="9ba29-118">Tailspin es la empresa ficticia que hospeda la aplicación Surveys.</span><span class="sxs-lookup"><span data-stu-id="9ba29-118">Tailspin is the fictitious company that hosts the Surveys application.</span></span> <span data-ttu-id="9ba29-119">Tailspin usa Azure AD para habilitar que otros inquilinos se registren con la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9ba29-119">Tailspin uses Azure AD to enable other tenants to register with the app.</span></span> <span data-ttu-id="9ba29-120">Luego, dichos clientes pueden usar sus credenciales de Azure AD para iniciar sesión en la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9ba29-120">Those customers can then use their Azure AD credentials to sign into the app.</span></span>

<span data-ttu-id="9ba29-121">En este paso, creará un directorio de Azure AD para Tailspin.</span><span class="sxs-lookup"><span data-stu-id="9ba29-121">In this step, you'll create an Azure AD directory for Tailspin.</span></span>

1. <span data-ttu-id="9ba29-122">Inicie sesión en [Azure Portal][portal].</span><span class="sxs-lookup"><span data-stu-id="9ba29-122">Sign into the [Azure portal][portal].</span></span>

2. <span data-ttu-id="9ba29-123">Haga clic en **Nuevo** > **Seguridad e identidad** > **Azure Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-123">Click **New** > **Security + Identity** > **Azure Active Directory**.</span></span>

3. <span data-ttu-id="9ba29-124">Escriba `Tailspin` como nombre de la organización y escriba un nombre de dominio.</span><span class="sxs-lookup"><span data-stu-id="9ba29-124">Enter `Tailspin` for the organization name, and enter a domain name.</span></span> <span data-ttu-id="9ba29-125">El nombre de dominio tendrá el formato `xxxx.onmicrosoft.com` y debe ser único globalmente.</span><span class="sxs-lookup"><span data-stu-id="9ba29-125">The domain name will have the form `xxxx.onmicrosoft.com` and must be globally unique.</span></span> 

    ![](./images/running-the-app/new-tenant.png)

4. <span data-ttu-id="9ba29-126">Haga clic en **Crear**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-126">Click **Create**.</span></span> <span data-ttu-id="9ba29-127">El nuevo directorio puede tardar unos minutos en crearse.</span><span class="sxs-lookup"><span data-stu-id="9ba29-127">It may take a few minutes to create the new directory.</span></span>

<span data-ttu-id="9ba29-128">Para completar el escenario de un extremo a otro, necesitará un segundo directorio de Azure AD para representar a un cliente que se suscribe a la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9ba29-128">To complete the end-to-end scenario, you'll need a second Azure AD directory to represent a customer that signs up for the application.</span></span> <span data-ttu-id="9ba29-129">Puede usar su directorio de Azure AD predeterminado (no Tailspin), o bien crear uno nuevo para este propósito.</span><span class="sxs-lookup"><span data-stu-id="9ba29-129">You can use your default Azure AD directory (not Tailspin), or create a new directory for this purpose.</span></span> <span data-ttu-id="9ba29-130">En los ejemplos, se usa Contoso como cliente ficticio.</span><span class="sxs-lookup"><span data-stu-id="9ba29-130">In the examples, we use Contoso as the fictitious customer.</span></span>

## <a name="register-the-surveys-web-api"></a><span data-ttu-id="9ba29-131">Registro de la API web de Surveys</span><span class="sxs-lookup"><span data-stu-id="9ba29-131">Register the Surveys web API</span></span> 

1. <span data-ttu-id="9ba29-132">En [Azure Portal][portal], cambie al nuevo directorio de Tailspin, para lo que debe seleccionar su cuenta en la esquina superior derecha del portal.</span><span class="sxs-lookup"><span data-stu-id="9ba29-132">In the [Azure portal][portal], switch to the new Tailspin directory by selecting your account in the top right corner of the portal.</span></span>

2. <span data-ttu-id="9ba29-133">En el panel de navegación izquierdo, elija **Azure Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-133">In the left-hand navigation pane, choose **Azure Active Directory**.</span></span> 

3. <span data-ttu-id="9ba29-134">Haga clic en **Registros de aplicaciones** > **Nuevo registro de aplicaciones**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-134">Click **App registrations** > **New application registration**.</span></span>

4.  <span data-ttu-id="9ba29-135">En la hoja **Crear**, escriba la siguiente información:</span><span class="sxs-lookup"><span data-stu-id="9ba29-135">In the **Create** blade, enter the following information:</span></span>

  - <span data-ttu-id="9ba29-136">**Nombre**: `Surveys.WebAPI`</span><span class="sxs-lookup"><span data-stu-id="9ba29-136">**Name**: `Surveys.WebAPI`</span></span>

  - <span data-ttu-id="9ba29-137">**Tipo de aplicación**: `Web app / API`</span><span class="sxs-lookup"><span data-stu-id="9ba29-137">**Application type**: `Web app / API`</span></span>

  - <span data-ttu-id="9ba29-138">**URL de inicio de sesión**: `https://localhost:44301/`</span><span class="sxs-lookup"><span data-stu-id="9ba29-138">**Sign-on URL**: `https://localhost:44301/`</span></span>
   
  ![](./images/running-the-app/register-web-api.png) 

5. <span data-ttu-id="9ba29-139">Haga clic en **Crear**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-139">Click **Create**.</span></span>

6. <span data-ttu-id="9ba29-140">En la hoja **Registros de aplicaciones**, seleccione la nueva aplicación **Surveys.WebAPI**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-140">In the **App registrations** blade, select the new **Surveys.WebAPI** application.</span></span>
 
7. <span data-ttu-id="9ba29-141">Haga clic en **Propiedades**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-141">Click **Properties**.</span></span>

8. <span data-ttu-id="9ba29-142">En el cuadro de edición **URI de id. de aplicación**, escriba `https://<domain>/surveys.webapi`, donde `<domain>` es el nombre de dominio del directorio.</span><span class="sxs-lookup"><span data-stu-id="9ba29-142">In the **App ID URI** edit box, enter `https://<domain>/surveys.webapi`, where `<domain>` is the domain name of the directory.</span></span> <span data-ttu-id="9ba29-143">Por ejemplo: `https://tailspin.onmicrosoft.com/surveys.webapi`</span><span class="sxs-lookup"><span data-stu-id="9ba29-143">For example: `https://tailspin.onmicrosoft.com/surveys.webapi`</span></span>

    ![Settings](./images/running-the-app/settings.png)

9. <span data-ttu-id="9ba29-145">Establecer **Multiinquilino** en **SÍ**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-145">Set **Multi-tenanted** to **YES**.</span></span>

10. <span data-ttu-id="9ba29-146">Haga clic en **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-146">Click **Save**.</span></span>

## <a name="register-the-surveys-web-app"></a><span data-ttu-id="9ba29-147">Registro de la aplicación web Surveys</span><span class="sxs-lookup"><span data-stu-id="9ba29-147">Register the Surveys web app</span></span> 

1.  <span data-ttu-id="9ba29-148">Vuelva a la hoja **Registros de aplicaciones** y haga clic en **Nuevo registro de aplicaciones**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-148">Navigate back to the **App registrations** blade, and click **New application registration**.</span></span>

2.  <span data-ttu-id="9ba29-149">En la hoja **Crear**, escriba la siguiente información:</span><span class="sxs-lookup"><span data-stu-id="9ba29-149">In the **Create** blade, enter the following information:</span></span>

  - <span data-ttu-id="9ba29-150">**Nombre**: `Surveys`</span><span class="sxs-lookup"><span data-stu-id="9ba29-150">**Name**: `Surveys`</span></span>
  - <span data-ttu-id="9ba29-151">**Tipo de aplicación**: `Web app / API`</span><span class="sxs-lookup"><span data-stu-id="9ba29-151">**Application type**: `Web app / API`</span></span>
  - <span data-ttu-id="9ba29-152">**URL de inicio de sesión**: `https://localhost:44300/`</span><span class="sxs-lookup"><span data-stu-id="9ba29-152">**Sign-on URL**: `https://localhost:44300/`</span></span>
   
    <span data-ttu-id="9ba29-153">Tenga en cuenta que la dirección URL de inicio de sesión tiene un número de puerto diferente que la aplicación `Surveys.WebAPI` del paso anterior.</span><span class="sxs-lookup"><span data-stu-id="9ba29-153">Notice that the sign-on URL has a different port number from the `Surveys.WebAPI` app in the previous step.</span></span>

3. <span data-ttu-id="9ba29-154">Haga clic en **Crear**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-154">Click **Create**.</span></span>
 
4. <span data-ttu-id="9ba29-155">En la hoja **Registros de aplicaciones**, seleccione la nueva aplicación **Surveys**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-155">In the **App registrations** blade, select the new **Surveys** application.</span></span>
 
5. <span data-ttu-id="9ba29-156">Copie el identificador de la aplicación.</span><span class="sxs-lookup"><span data-stu-id="9ba29-156">Copy the application ID.</span></span> <span data-ttu-id="9ba29-157">Lo necesitará más adelante.</span><span class="sxs-lookup"><span data-stu-id="9ba29-157">You will need this later.</span></span>

    ![](./images/running-the-app/application-id.png)

6. <span data-ttu-id="9ba29-158">Haga clic en **Propiedades**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-158">Click **Properties**.</span></span>

7. <span data-ttu-id="9ba29-159">En el cuadro de edición **URI de id. de aplicación**, escriba `https://<domain>/surveys`, donde `<domain>` es el nombre de dominio del directorio.</span><span class="sxs-lookup"><span data-stu-id="9ba29-159">In the **App ID URI** edit box, enter `https://<domain>/surveys`, where `<domain>` is the domain name of the directory.</span></span> 

    ![Settings](./images/running-the-app/settings.png)

8. <span data-ttu-id="9ba29-161">Establecer **Multiinquilino** en **SÍ**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-161">Set **Multi-tenanted** to **YES**.</span></span>

9. <span data-ttu-id="9ba29-162">Haga clic en **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-162">Click **Save**.</span></span>

10. <span data-ttu-id="9ba29-163">En la hoja **Configuración**, haga clic en **URL de respuesta**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-163">In the **Settings** blade, click **Reply URLs**.</span></span>
 
11. <span data-ttu-id="9ba29-164">Agregue la siguiente dirección URL de respuesta: `https://localhost:44300/signin-oidc`.</span><span class="sxs-lookup"><span data-stu-id="9ba29-164">Add the following reply URL: `https://localhost:44300/signin-oidc`.</span></span>

12. <span data-ttu-id="9ba29-165">Haga clic en **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-165">Click **Save**.</span></span>

13. <span data-ttu-id="9ba29-166">En **ACCESO DE API**, elija **Claves**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-166">Under **API ACCESS**, click **Keys**.</span></span>

14. <span data-ttu-id="9ba29-167">Escriba una descripción, como `client secret`.</span><span class="sxs-lookup"><span data-stu-id="9ba29-167">Enter a description, such as `client secret`.</span></span>

15. <span data-ttu-id="9ba29-168">En la lista desplegable **Seleccionar duración**, seleccione **1 año**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-168">In the **Select Duration** dropdown, select **1 year**.</span></span> 

16. <span data-ttu-id="9ba29-169">Haga clic en **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-169">Click **Save**.</span></span> <span data-ttu-id="9ba29-170">La clave se generará al guardar.</span><span class="sxs-lookup"><span data-stu-id="9ba29-170">The key will be generated when you save.</span></span>

17. <span data-ttu-id="9ba29-171">Antes de salir de esta hoja, copie el valor de la clave.</span><span class="sxs-lookup"><span data-stu-id="9ba29-171">Before you navigate away from this blade, copy the value of the key.</span></span>

    > [!NOTE] 
    > <span data-ttu-id="9ba29-172">La clave no volverá a estará visible después de salir de la hoja.</span><span class="sxs-lookup"><span data-stu-id="9ba29-172">The key won't be visible again after you navigate away from the blade.</span></span> 

18. <span data-ttu-id="9ba29-173">En **ACCESO DE API**, haga clic en **Permisos necesarios**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-173">Under **API ACCESS**, click **Required permissions**.</span></span>

19. <span data-ttu-id="9ba29-174">Haga clic en **Agregar** > **Seleccionar una API**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-174">Click **Add** > **Select an API**.</span></span>

20. <span data-ttu-id="9ba29-175">En el cuadro de búsqueda, escriba `Surveys.WebAPI`.</span><span class="sxs-lookup"><span data-stu-id="9ba29-175">In the search box, search for `Surveys.WebAPI`.</span></span>

    ![Permisos](./images/running-the-app/permissions.png)

21. <span data-ttu-id="9ba29-177">Seleccione `Surveys.WebAPI` y haga clic en **Seleccionar**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-177">Select `Surveys.WebAPI` and click **Select**.</span></span>

22. <span data-ttu-id="9ba29-178">En **Permisos delegados**, seleccione **Access Surveys.WebAPI**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-178">Under **Delegated Permissions**, check **Access Surveys.WebAPI**.</span></span>

    ![Establecimiento de permisos delegados](./images/running-the-app/delegated-permissions.png)

23. <span data-ttu-id="9ba29-180">Haga clic en **Seleccionar** > **Listo**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-180">Click **Select** > **Done**.</span></span>


## <a name="update-the-application-manifests"></a><span data-ttu-id="9ba29-181">Actualización de los manifiestos de aplicaciones</span><span class="sxs-lookup"><span data-stu-id="9ba29-181">Update the application manifests</span></span>

1. <span data-ttu-id="9ba29-182">Vuelva a la hoja **Configuración** de la aplicación `Surveys.WebAPI`.</span><span class="sxs-lookup"><span data-stu-id="9ba29-182">Navigate back to the **Settings** blade for the `Surveys.WebAPI` app.</span></span>

2. <span data-ttu-id="9ba29-183">Haga clic en **Manifiesto** > **Editar**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-183">Click **Manifest** > **Edit**.</span></span>

    ![](./images/running-the-app/manifest.png)
 
3.  <span data-ttu-id="9ba29-184">Agregue el siguiente JSON al elemento `appRoles`.</span><span class="sxs-lookup"><span data-stu-id="9ba29-184">Add the following JSON to the `appRoles` element.</span></span> <span data-ttu-id="9ba29-185">Genere nuevos GUID para las propiedades `id`.</span><span class="sxs-lookup"><span data-stu-id="9ba29-185">Generate new GUIDs for the `id` properties.</span></span>

    ```json
    {
      "allowedMemberTypes": ["User"],
      "description": "Creators can create surveys",
      "displayName": "SurveyCreator",
      "id": "<Generate a new GUID. Example: 1b4f816e-5eaf-48b9-8613-7923830595ad>",
      "isEnabled": true,
      "value": "SurveyCreator"
    },
    {
      "allowedMemberTypes": ["User"],
      "description": "Administrators can manage the surveys in their tenant",
      "displayName": "SurveyAdmin",
      "id": "<Generate a new GUID>",  
      "isEnabled": true,
      "value": "SurveyAdmin"
    }
    ```

5.  <span data-ttu-id="9ba29-186">En la propiedad `knownClientApplications`, agregue el identificador de aplicación de la aplicación web Surveys, que obtuvo al registrar la aplicación anteriormente.</span><span class="sxs-lookup"><span data-stu-id="9ba29-186">In the `knownClientApplications` property, add the application ID for the Surveys web application, which you got when you registered the Surveys application earlier.</span></span> <span data-ttu-id="9ba29-187">Por ejemplo:</span><span class="sxs-lookup"><span data-stu-id="9ba29-187">For example:</span></span>

  ```json
  "knownClientApplications": ["be2cea23-aa0e-4e98-8b21-2963d494912e"],
  ```

  <span data-ttu-id="9ba29-188">Esta opción agrega la aplicación Surveys a la lista de clientes autorizados para llamar a la API web.</span><span class="sxs-lookup"><span data-stu-id="9ba29-188">This setting adds the Surveys app to the list of clients authorized to call the web API.</span></span>

6.  <span data-ttu-id="9ba29-189">Haga clic en **Guardar**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-189">Click **Save**.</span></span>

<span data-ttu-id="9ba29-190">Ahora repita los mismos pasos para la aplicación Surveys, pero no agregue una entrada para `knownClientApplications`.</span><span class="sxs-lookup"><span data-stu-id="9ba29-190">Now repeat the same steps for the Surveys app, except do not add an entry for `knownClientApplications`.</span></span> <span data-ttu-id="9ba29-191">Use las mismas definiciones de rol, pero genere nuevos GUID para los identificadores.</span><span class="sxs-lookup"><span data-stu-id="9ba29-191">Use the same role definitions, but generate new GUIDs for the IDs.</span></span>

## <a name="create-a-new-redis-cache-instance"></a><span data-ttu-id="9ba29-192">Creación una instancia nueva de Redis Cache</span><span class="sxs-lookup"><span data-stu-id="9ba29-192">Create a new Redis Cache instance</span></span>

<span data-ttu-id="9ba29-193">La aplicación Surveys utiliza Redis para almacenar en caché tokens de acceso de OAuth 2.</span><span class="sxs-lookup"><span data-stu-id="9ba29-193">The Surveys application uses Redis to cache OAuth 2 access tokens.</span></span> <span data-ttu-id="9ba29-194">Para crear la caché:</span><span class="sxs-lookup"><span data-stu-id="9ba29-194">To create the cache:</span></span>

1.  <span data-ttu-id="9ba29-195">Vaya a [Azure Portal](https://portal.azure.com) y haga clic en **Nuevo** > **Databases** > **Caché en Redis**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-195">Go to [Azure Portal](https://portal.azure.com) and click **New** > **Databases** > **Redis Cache**.</span></span>

2.  <span data-ttu-id="9ba29-196">Rellene la información necesaria, incluidos el nombre DNS, el grupo de recursos, la ubicación y el plan de tarifa.</span><span class="sxs-lookup"><span data-stu-id="9ba29-196">Fill in the required information, including DNS name, resource group, location, and pricing tier.</span></span> <span data-ttu-id="9ba29-197">Puede crear un grupo de recursos nuevo o seleccionar uno existente.</span><span class="sxs-lookup"><span data-stu-id="9ba29-197">You can create a new resource group or use an existing resource group.</span></span>

3. <span data-ttu-id="9ba29-198">Haga clic en **Crear**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-198">Click **Create**.</span></span>

4. <span data-ttu-id="9ba29-199">Después de crea la caché en Resis, navegue hasta el recurso en el portal.</span><span class="sxs-lookup"><span data-stu-id="9ba29-199">After the Resis cache is created, navigate to the resource in the portal.</span></span>

5. <span data-ttu-id="9ba29-200">Haga clic en **Claves de acceso** y copie la clave principal.</span><span class="sxs-lookup"><span data-stu-id="9ba29-200">Click **Access keys** and copy the primary key.</span></span>

<span data-ttu-id="9ba29-201">Para más información acerca de cómo crear una caché en Redis, consulte [Uso de Azure Redis Cache](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache).</span><span class="sxs-lookup"><span data-stu-id="9ba29-201">For more information about creating a Redis cache, see [How to Use Azure Redis Cache](/azure/redis-cache/cache-dotnet-how-to-use-azure-redis-cache).</span></span>

## <a name="set-application-secrets"></a><span data-ttu-id="9ba29-202">Establecimiento de secretos de la aplicación</span><span class="sxs-lookup"><span data-stu-id="9ba29-202">Set application secrets</span></span>

1.  <span data-ttu-id="9ba29-203">Abra la solución Tailspin.Surveys en Visual Studio.</span><span class="sxs-lookup"><span data-stu-id="9ba29-203">Open the Tailspin.Surveys solution in Visual Studio.</span></span>

2.  <span data-ttu-id="9ba29-204">En el Explorador de soluciones, haga clic con el botón derecho en el proyecto Tailspin.Surveys.Web y seleccione **Administrar secretos de usuario**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-204">In Solution Explorer, right-click the Tailspin.Surveys.Web project and select **Manage User Secrets**.</span></span>

3.  <span data-ttu-id="9ba29-205">En el archivo secrets.json, pegue lo siguiente:</span><span class="sxs-lookup"><span data-stu-id="9ba29-205">In the secrets.json file, paste in the following:</span></span>
    
    ```json
    {
      "AzureAd": {
        "ClientId": "<Surveys application ID>",
        "ClientSecret": "<Surveys app client secret>",
        "PostLogoutRedirectUri": "https://localhost:44300/",
        "WebApiResourceId": "<Surveys.WebAPI app ID URI>"
      },
      "Redis": {
        "Configuration": "<Redis DNS name>.redis.cache.windows.net,password=<Redis primary key>,ssl=true"
      }
    }
    ```
   
    <span data-ttu-id="9ba29-206">Reemplace los elementos que aparecen entre corchetes angulares como se indica a continuación:</span><span class="sxs-lookup"><span data-stu-id="9ba29-206">Replace the items shown in angle brackets, as follows:</span></span>

    - <span data-ttu-id="9ba29-207">`AzureAd:ClientId`: el identificador de la aplicación Surveys.</span><span class="sxs-lookup"><span data-stu-id="9ba29-207">`AzureAd:ClientId`: The application ID of the Surveys app.</span></span>
    - <span data-ttu-id="9ba29-208">`AzureAd:ClientSecret`: la clave que generó al registrar la aplicación Surveys en Azure AD.</span><span class="sxs-lookup"><span data-stu-id="9ba29-208">`AzureAd:ClientSecret`: The key that you generated when you registered the Surveys application in Azure AD.</span></span>
    - <span data-ttu-id="9ba29-209">`AzureAd:WebApiResourceId`: URI del identificador de aplicación que especificó cuando creó la aplicación Surveys.WebAPI en Azure AD.</span><span class="sxs-lookup"><span data-stu-id="9ba29-209">`AzureAd:WebApiResourceId`: The App ID URI that you specified when you created the Surveys.WebAPI application in Azure AD.</span></span> <span data-ttu-id="9ba29-210">Debe tener el formato `https://<directory>.onmicrosoft.com/surveys.webapi`</span><span class="sxs-lookup"><span data-stu-id="9ba29-210">It should have the form `https://<directory>.onmicrosoft.com/surveys.webapi`</span></span>
    - <span data-ttu-id="9ba29-211">`Redis:Configuration`: cree esta cadena a partir del nombre DNS de la caché en Redis y la clave de acceso principal.</span><span class="sxs-lookup"><span data-stu-id="9ba29-211">`Redis:Configuration`: Build this string from the DNS name of the Redis cache and the primary access key.</span></span> <span data-ttu-id="9ba29-212">Por ejemplo, "tailspin.redis.cache.windows.net,password=2h5tBxxx,ssl=true".</span><span class="sxs-lookup"><span data-stu-id="9ba29-212">For example, "tailspin.redis.cache.windows.net,password=2h5tBxxx,ssl=true".</span></span>

4.  <span data-ttu-id="9ba29-213">Guarde el archivo actualizado secrets.json.</span><span class="sxs-lookup"><span data-stu-id="9ba29-213">Save the updated secrets.json file.</span></span>

5.  <span data-ttu-id="9ba29-214">Repita estos pasos para el proyecto Tailspin.Surveys.WebAPI y pegue lo siguiente en secrets.json.</span><span class="sxs-lookup"><span data-stu-id="9ba29-214">Repeat these steps for the Tailspin.Surveys.WebAPI project, but paste the following into secrets.json.</span></span> <span data-ttu-id="9ba29-215">Reemplace los elementos entre corchetes angulares como se ha indicado.</span><span class="sxs-lookup"><span data-stu-id="9ba29-215">Replace the items in angle brackets, as before.</span></span>

    ```json
    {
      "AzureAd": {
        "WebApiResourceId": "<Surveys.WebAPI app ID URI>"
      },
      "Redis": {
        "Configuration": "<Redis DNS name>.redis.cache.windows.net,password=<Redis primary key>,ssl=true"
      }
    }
    ```

## <a name="initialize-the-database"></a><span data-ttu-id="9ba29-216">Inicialización de la base de datos</span><span class="sxs-lookup"><span data-stu-id="9ba29-216">Initialize the database</span></span>

<span data-ttu-id="9ba29-217">En este paso, usará Entity Framework 7 para crear una base de datos SQL local, con LocalDB.</span><span class="sxs-lookup"><span data-stu-id="9ba29-217">In this step, you will use Entity Framework 7 to create a local SQL database, using LocalDB.</span></span>

1.  <span data-ttu-id="9ba29-218">Abra una ventana de comando</span><span class="sxs-lookup"><span data-stu-id="9ba29-218">Open a command window</span></span>

2.  <span data-ttu-id="9ba29-219">Navegue hasta el proyecto Tailspin.Surveys.Data.</span><span class="sxs-lookup"><span data-stu-id="9ba29-219">Navigate to the Tailspin.Surveys.Data project.</span></span>

3.  <span data-ttu-id="9ba29-220">Ejecute el siguiente comando:</span><span class="sxs-lookup"><span data-stu-id="9ba29-220">Run the following command:</span></span>

    ```
    dotnet ef database update --startup-project ..\Tailspin.Surveys.Web
    ```
    
## <a name="run-the-application"></a><span data-ttu-id="9ba29-221">Ejecución de la aplicación</span><span class="sxs-lookup"><span data-stu-id="9ba29-221">Run the application</span></span>

<span data-ttu-id="9ba29-222">Para ejecutar la aplicación, inicie los proyectos Tailspin.Surveys.Web y Tailspin.Surveys.WebAPI.</span><span class="sxs-lookup"><span data-stu-id="9ba29-222">To run the application, start both the Tailspin.Surveys.Web and Tailspin.Surveys.WebAPI projects.</span></span>

<span data-ttu-id="9ba29-223">Puede establecer que Visual Studio se ejecute en ambos proyectos automáticamente en F5, como se indica a continuación:</span><span class="sxs-lookup"><span data-stu-id="9ba29-223">You can set Visual Studio to run both projects automatically on F5, as follows:</span></span>

1.  <span data-ttu-id="9ba29-224">En el Explorador de soluciones, haga clic con el botón derecho en la solución y después en **Establecer proyectos de inicio**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-224">In Solution Explorer, right-click the solution and click **Set Startup Projects**.</span></span>
2.  <span data-ttu-id="9ba29-225">Seleccione **Proyectos de inicio múltiples**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-225">Select **Multiple startup projects**.</span></span>
3.  <span data-ttu-id="9ba29-226">Establezca **Acción** = **Iniciar** para los proyectos Tailspin.Surveys.Web y Tailspin.Surveys.WebAPI.</span><span class="sxs-lookup"><span data-stu-id="9ba29-226">Set **Action** = **Start** for the Tailspin.Surveys.Web and Tailspin.Surveys.WebAPI projects.</span></span>

## <a name="sign-up-a-new-tenant"></a><span data-ttu-id="9ba29-227">Suscripción de un nuevo inquilino</span><span class="sxs-lookup"><span data-stu-id="9ba29-227">Sign up a new tenant</span></span>

<span data-ttu-id="9ba29-228">Cuando la aplicación se inicia, no ha iniciado sesión, por lo que verá la página de bienvenida:</span><span class="sxs-lookup"><span data-stu-id="9ba29-228">When the application starts, you are not signed in, so you see the welcome page:</span></span>

![Página de bienvenida](./images/running-the-app/screenshot1.png)

<span data-ttu-id="9ba29-230">Para registrar una organización:</span><span class="sxs-lookup"><span data-stu-id="9ba29-230">To sign up an organization:</span></span>

1. <span data-ttu-id="9ba29-231">Haga clic en **Enroll your company in Tailspin** (Inscribir su compañía en Tailspin).</span><span class="sxs-lookup"><span data-stu-id="9ba29-231">Click **Enroll your company in Tailspin**.</span></span>
2. <span data-ttu-id="9ba29-232">Inicie sesión en el directorio de Azure AD que representa la organización mediante la aplicación Surveys.</span><span class="sxs-lookup"><span data-stu-id="9ba29-232">Sign in to the Azure AD directory that represents the organization using the Surveys app.</span></span> <span data-ttu-id="9ba29-233">Debe iniciar sesión como un usuario administrador.</span><span class="sxs-lookup"><span data-stu-id="9ba29-233">You must sign in as an admin user.</span></span>
3. <span data-ttu-id="9ba29-234">Acepte la petición de consentimiento.</span><span class="sxs-lookup"><span data-stu-id="9ba29-234">Accept the consent prompt.</span></span>

<span data-ttu-id="9ba29-235">La aplicación registra al inquilino y, después, cierra la sesión. La aplicación cierra la sesión porque antes de usarla hay que configurar los roles de aplicación en Azure AD.</span><span class="sxs-lookup"><span data-stu-id="9ba29-235">The application registers the tenant, and then signs you out. The app signs you out because you need to set up the application roles in Azure AD, before using the application.</span></span>

![Después de la suscripción](./images/running-the-app/screenshot2.png)

## <a name="assign-application-roles"></a><span data-ttu-id="9ba29-237">Asignación de los roles de la aplicación</span><span class="sxs-lookup"><span data-stu-id="9ba29-237">Assign application roles</span></span>

<span data-ttu-id="9ba29-238">Cuando un inquilino se suscribe, un administrador de AD del inquilino debe asignar roles de aplicación a los usuarios.</span><span class="sxs-lookup"><span data-stu-id="9ba29-238">When a tenant signs up, an AD admin for the tenant must assign application roles to users.</span></span>


1. <span data-ttu-id="9ba29-239">En el [Azure Portal][portal], cambie al directorio de Azure AD que usó para suscribirse a la aplicación Surveys.</span><span class="sxs-lookup"><span data-stu-id="9ba29-239">In the [Azure portal][portal], switch to the Azure AD directory that you used to sign up for the Surveys app.</span></span> 

2. <span data-ttu-id="9ba29-240">En el panel de navegación izquierdo, elija **Azure Active Directory**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-240">In the left-hand navigation pane, choose **Azure Active Directory**.</span></span> 

3. <span data-ttu-id="9ba29-241">Haga clic en **Aplicaciones empresariales** > **Todas las aplicaciones**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-241">Click **Enterprise applications** > **All applications**.</span></span> <span data-ttu-id="9ba29-242">El portal enumerará `Survey` y `Survey.WebAPI`.</span><span class="sxs-lookup"><span data-stu-id="9ba29-242">The portal will list `Survey` and `Survey.WebAPI`.</span></span> <span data-ttu-id="9ba29-243">Si no lo hace, asegúrese de que ha completado el proceso de suscripción.</span><span class="sxs-lookup"><span data-stu-id="9ba29-243">If not, make sure that you completed the sign up process.</span></span>

4.  <span data-ttu-id="9ba29-244">Haga clic en la aplicación Surveys.</span><span class="sxs-lookup"><span data-stu-id="9ba29-244">Click on the Surveys application.</span></span>

5.  <span data-ttu-id="9ba29-245">Haga clic en **Usuarios y grupos**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-245">Click **Users and Groups**.</span></span>

4.  <span data-ttu-id="9ba29-246">Haga clic en **Agregar usuario**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-246">Click **Add user**.</span></span>

5.  <span data-ttu-id="9ba29-247">Si dispone de Azure AD Premium, haga clic en **Usuarios y grupos**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-247">If you have Azure AD Premium, click **Users and groups**.</span></span> <span data-ttu-id="9ba29-248">De lo contrario, haga clic en **Usuarios**</span><span class="sxs-lookup"><span data-stu-id="9ba29-248">Otherwise, click **Users**.</span></span> <span data-ttu-id="9ba29-249">(la asignación de un rol a un grupo requiere Azure AD Premium).</span><span class="sxs-lookup"><span data-stu-id="9ba29-249">(Assigning a role to a group requires Azure AD Premium.)</span></span>

6. <span data-ttu-id="9ba29-250">Seleccione uno o varios usuarios y haga clic en **Seleccionar**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-250">Select one or more users and click **Select**.</span></span>

    ![Seleccionar usuario o grupo](./images/running-the-app/select-user-or-group.png)

6.  <span data-ttu-id="9ba29-252">Seleccione el rol y haga clic en **Seleccionar**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-252">Select the role and click **Select**.</span></span>

    ![Seleccionar usuario o grupo](./images/running-the-app/select-role.png)

7.  <span data-ttu-id="9ba29-254">Haga clic en **Asignar**.</span><span class="sxs-lookup"><span data-stu-id="9ba29-254">Click **Assign**.</span></span>

<span data-ttu-id="9ba29-255">Repita los mismos pasos para asignar roles de la aplicación Survey.WebAPI.</span><span class="sxs-lookup"><span data-stu-id="9ba29-255">Repeat the same steps to assign roles for the Survey.WebAPI application.</span></span>

> <span data-ttu-id="9ba29-256">Importante: Un usuario siempre debe tener los mismos roles en Survey y en Survey.WebAPI.</span><span class="sxs-lookup"><span data-stu-id="9ba29-256">Important: A user should always have the same roles in both Survey and Survey.WebAPI.</span></span> <span data-ttu-id="9ba29-257">De lo contrario, los permisos del usuario no serán coherentes, lo que puede provocar errores de 403 (Prohibido) de la API Web.</span><span class="sxs-lookup"><span data-stu-id="9ba29-257">Otherwise, the user will have inconsistent permissions, which may lead to 403 (Forbidden) errors from the Web API.</span></span>

<span data-ttu-id="9ba29-258">Vuelva a la aplicación e inicie sesión de otra vez.</span><span class="sxs-lookup"><span data-stu-id="9ba29-258">Now go back to the app and sign in again.</span></span> <span data-ttu-id="9ba29-259">Haga clic en **My Surveys** (Mis encuestas).</span><span class="sxs-lookup"><span data-stu-id="9ba29-259">Click **My Surveys**.</span></span> <span data-ttu-id="9ba29-260">Si al usuario se le asigna el rol SurveyAdmin o SurveyCreator, verá un botón **Create Survey** (Crear encuesta), que indica que el usuario tiene permisos para crear una nueva encuesta.</span><span class="sxs-lookup"><span data-stu-id="9ba29-260">If the user is assigned to the SurveyAdmin or SurveyCreator role, you will see a **Create Survey** button, indicating that the user has permissions to create a new survey.</span></span>

![My surveys (Mis encuestas)](./images/running-the-app/screenshot3.png)


<!-- links -->

[portal]: https://portal.azure.com
[VS2017]: https://www.visualstudio.com/vs/
