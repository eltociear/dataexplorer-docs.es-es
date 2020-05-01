---
title: 'HowTo: creación de una aplicación de AAD: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe cómo crear una aplicación de AAD en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/18/2020
ms.openlocfilehash: 0816d55cda6d29820084514b0bfec27d726693f9
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617976"
---
# <a name="howto-creating-an-aad-application"></a>Cómo: crear una aplicación de AAD

Aunque la autenticación de usuario de AAD es muy sencilla (dado que el administrador del inquilino, como MSIT), la autenticación de la aplicación de AAD es algo más compleja, ya que los usuarios se definen en AAD. Esto se debe a que requiere la creación y el registro de la aplicación con AAD. Este proceso se describe a continuación en algunos detalles.

La autenticación de aplicaciones de AAD es útil para las aplicaciones que necesitan acceder a Kusto sin que un usuario inicie sesión o presente (por ejemplo, un servicio desatendido o un flujo programado).

Las aplicaciones cliente y las aplicaciones de nivel intermedio que tienen contexto de usuario interactivo deben evitar este modelo. Esto se debe a que la autorización se realiza en función de la identidad de la aplicación de AAD en lugar de la identidad del usuario, por lo que la aplicación que realiza la llamada deberá implementar su propia lógica de autorización para evitar el uso incorrecto.

## <a name="application-authentication-use-cases"></a>Casos de uso de autenticación de aplicaciones

Existen dos escenarios principales que hacen uso de la autenticación de la aplicación:
* Aplicaciones que se pongan en contacto directamente con el servicio Kusto y en su propio nombre
* Aplicaciones que autenticarán a sus usuarios en Kusto (autenticación delegada)

## <a name="1-provisioning-a-new-application"></a>1. aprovisionamiento de una nueva aplicación

#### <a name="register-the-new-application"></a>Registrar la nueva aplicación

1. Inicie sesión en [Azure portal](https://portal.azure.com) y abra la `Azure Active Directory` hoja.

    :::image type="content" source="images/aad-create-app-step-0.png" alt-text="Paso 0 de creación de aplicación de AAD":::

1. Elija `App registrations` y cuando se cargue la hoja, `New application registration`seleccione.

    :::image type="content" source="images/aad-create-app-step-1.png" alt-text="Paso 1 de creación de aplicación de AAD":::

1. Rellene los detalles de la aplicación:
    * Nombre
    * Tipo: establézcalo en`Web app/API`
    * URL de inicio de sesión: la dirección URL que usan los usuarios para tener acceso a la aplicación. AAD no valida la dirección URL,<br>
        pero le asigna un valor. Por tanto, si la aplicación no tiene ninguna dirección URL accesible para los usuarios,<br>
        Escriba una dirección URL que pertenezca a la aplicación (por ejemplo, https://<APP-CNAME> o https://<CLOUD-SERVICE-NAME>. cloudapp.net).<br>
        Incluso puede proporcionar un valor y continuar si aún no se ha escrito la aplicación.

    
    :::image type="content" source="images/aad-create-app-step-2.png" alt-text="Paso 2 de creación de aplicación de AAD":::

1. Una vez que la aplicación esté lista, `Settings` Abra su hoja.

    :::image type="content" source="images/aad-create-app-step-3.png" alt-text="Creación de AAD, paso 3":::

1. En la `Keys` hoja, configure la autenticación para la nueva aplicación:
    * Si desea usar una clave compartida, seleccione Key Duration (duración de clave) en el menú desplegable y copie la clave cuando se genere.
        No podrá restaurar esta clave.
    * También puede usar un certificado X509 para autenticar la aplicación.
        Para ello, seleccione `Upload Public Key` y siga las instrucciones para cargar la parte pública del certificado.
    * No olvide seleccionar `Save` cuando haya terminado.

    :::image type="content" source="images/aad-create-app-step-4.png" alt-text="Paso 4 de creación de aplicación de AAD":::

La aplicación está configurada. Si todo lo que necesita es el acceso a Kusto con la aplicación recién creada, ha terminado.

#### <a name="set-up-delegated-permissions-for-kusto-service-application"></a>Configurar permisos delegados para la aplicación de servicio Kusto

Si necesita que la aplicación pueda realizar la autenticación de usuarios o aplicaciones para el acceso de Kusto, debe configurar la confianza entre la aplicación y Kusto:

1. En la `Required permissions` hoja, seleccione `Add`.

    :::image type="content" source="images/aad-create-app-step-5.png" alt-text="Paso 5 de creación de aplicación de AAD":::
   
1. Elija `Select an API`, escriba `KustoService` en el cuadro de filtro y `KustoService (Kusto)`seleccione.
1. Si los clústeres de Kusto requieren MFA, `KustoMFA` escriba en el cuadro de filtro `KustoServiceMFA (KustoMFA)`y seleccione.

    :::image type="content" source="images/aad-create-app-step-6.png" alt-text="Paso 6 de creación de aplicación de AAD":::

1. Después de confirmar la selección, elija permisos delegados para `Access Kusto`.

   :::image type="content" source="images/aad-create-app-step-7.png" alt-text="Paso 7 de creación de aplicación de AAD":::

1. Seleccione `Done` esta función para completar el proceso.


### <a name="2-set-permissions-to-the-application-on-kusto-cluster"></a>2. establecer permisos para la aplicación en el clúster de Kusto

> Antes de usar la aplicación recién aprovisionada, compruebe que se resuelve desde Graph API.<br>
    Los cambios de AAD pueden tardar hasta varias horas en propagarse.

1. Obtenga acceso al administrador de la base de datos para conceder permisos a la nueva aplicación.
Consulte la sección [Administración de entidades de seguridad de base de datos](../security-roles.md) para obtener información detallada sobre cómo establecer los permisos.<br>
Para configurar los permisos de ingesta, vea [permisos de ingesta](../../api/netfx/kusto-ingest-client-permissions.md).
1. Agregue una conexión a este servicio en el explorador de Kusto, si aún no lo tiene.

### <a name="3-application-can-now-access-kusto"></a>3. la aplicación ahora puede acceder a Kusto

Al usar la aplicación recién aprovisionada para acceder a los clústeres de Kusto mediante bibliotecas de cliente de Kusto, especifique la cadena de conexión siguiente (si decide autenticarse con una clave compartida):

`https://`*ClusterDnsName*`/;Federated Security=True;Application Client Id=`*ApplicationClientId*ApplicationClientId`;Application Key=`*ApplicationKey*


### <a name="appendix-a-aad-errors"></a>Apéndice A: errores de AAD

#### <a name="aad-error-aadsts650057"></a>Error AADSTS650057 de AAD

Si la aplicación se usa para autenticar a usuarios o aplicaciones para el acceso de Kusto, debe configurar permisos delegados para la aplicación de servicio de Kusto, es decir, declarar que la aplicación puede autenticar a usuarios o aplicaciones para el acceso a Kusto.
Si no lo hace, se producirá un error similar al siguiente cuando se realice un intento de autenticación:

`AADSTS650057: Invalid resource. The client has requested access to a resource which is not listed in the requested permissions in the client's application registration...`

Tendrá que seguir las instrucciones para [configurar permisos delegados para la aplicación de servicio Kusto](#set-up-delegated-permissions-for-kusto-service-application).

#### <a name="enable-user-consent-if-needed"></a>Habilitar el consentimiento del usuario si es necesario

El administrador de inquilinos de AAD puede adoptar una directiva que impida a los usuarios del inquilino dar su consentimiento a las aplicaciones. Esto producirá un error similar al siguiente, cuando un usuario intente iniciar sesión en su aplicación:

`AADSTS65001: The user or administrator has not consented to use the application with ID '<App ID>' named 'App Name'`

Tendrá que pedir al administrador de AAD que conceda su consentimiento a todos los usuarios del inquilino, o que habilite el consentimiento del usuario para la aplicación específica.

### <a name="appendix-b-advanced-topics-and-troubleshooting"></a>Apéndice B: temas avanzados y solución de problemas

* Consulte la documentación de [cadenas de conexión de Kusto](../../api/connection-strings/kusto.md) para ver la referencia completa de las cadenas de conexión admitidas.
