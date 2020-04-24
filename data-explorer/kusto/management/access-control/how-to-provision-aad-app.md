---
title: HowTo - Creación de una aplicación de AAD - Explorador de azure Data Explorer Microsoft Docs
description: 'En este artículo se describe HowTo: creación de una aplicación de AAD en Azure Data Explorer.'
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/18/2020
ms.openlocfilehash: 91d646d3bd0922f9daf9e8caec6fa6be5e32e2b7
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522906"
---
# <a name="howto-creating-an-aad-application"></a>Cómo: Creación de una aplicación de AAD

Aunque la autenticación de usuario de AAD es muy fácil (porque los usuarios están definidos en AAD por el administrador de inquilinos, como MSIT), la autenticación de aplicaciones de AAD es algo más compleja. Esto se debe a que requiere que cree y registre la aplicación con AAD. Este proceso se describe a continuación en algunos detalles.

La autenticación de aplicaciones de AAD es útil para las aplicaciones que necesitan acceder a Kusto sin que un usuario esté conectado o presente (por ejemplo, un servicio desatendido o un flujo programado).

Las aplicaciones cliente y las aplicaciones de nivel intermedio que tienen contexto de usuario interactivo deben evitar este modelo. Esto se debe a que la autorización se realiza en función de la identidad de la aplicación de AAD en lugar de la identidad del usuario, por lo que la aplicación que realiza la llamada tendrá que implementar su propia lógica de autorización para evitar el uso indebido.

## <a name="application-authentication-use-cases"></a>Casos de uso de autenticación de aplicaciones

Hay dos escenarios principales que hacen uso de la autenticación de aplicaciones:
* Aplicaciones que se contactan con el servicio de Kusto directamente y en su propio nombre
* Aplicaciones que autenticarán a sus usuarios en Kusto (autenticación delegada)

## <a name="1-provisioning-a-new-application"></a>1. Aprovisionamiento de una nueva aplicación

#### <a name="register-the-new-application"></a>Registrar la nueva aplicación

1. Inicie sesión en Azure `Azure Active Directory` [Portal](https://portal.azure.com) y abra la hoja.

    ![texto alternativo](./images/Aad-create-app-step-0.png "Aad-create-app-step-0")

1. Elija `App registrations` y cuando se `New application registration`cargue la hoja, seleccione .

    ![texto alternativo](./images/Aad-create-app-step-1.png "Aad-create-app-step-1")

1. Rellene los detalles de la solicitud:
    * NOMBRE
    * Tipo: establecido en`Web app/API`
    * URL de inicio de sesión: la dirección URL utilizada por los usuarios para acceder a la aplicación. AAD no valida la dirección URL,<br>
        pero exige que usted proporcione un valor. Así que si la aplicación no tiene ninguna URL accesible para los usuarios,<br>
        escriba una dirección URL que pertenezca a la aplicación (como https://<APP-CNAME> o https://<CLOUD-SERVICE-NAME>.cloudapp.net).<br>
        Incluso puede proporcionar un valor y continuar si la aplicación aún no se ha escrito.

    ![texto alternativo](./images/Aad-create-app-step-2.png "Aad-create-app-step-2")

1. Una vez que la `Settings` aplicación esté lista, abra su hoja.

    ![texto alternativo](./images/Aad-create-app-step-3.png "Aad-create-app-step-3")

1. En `Keys` la hoja, configure la autenticación para la nueva aplicación:
    * Si desea utilizar una clave compartida, seleccione la duración de la clave en el menú desplegable y copie la clave cuando se genere.
        No podrá restaurar esta clave.
    * Como alternativa, utilice un certificado X509 para autenticar la aplicación.
        Para ello, `Upload Public Key` seleccione y siga las instrucciones para cargar la parte pública del certificado.
    * No olvides seleccionar `Save` cuando termines.

    ![texto alternativo](./images/Aad-create-app-step-4.png "Aad-create-app-step-4")

La aplicación está configurada. Si todo lo que necesita es acceso a Kusto con la aplicación recién creada, ya está.

#### <a name="set-up-delegated-permissions-for-kusto-service-application"></a>Configurar permisos delegados para la aplicación de servicio Kusto

Si necesita que la aplicación pueda realizar la autenticación de usuario o aplicación para el acceso a Kusto, debe configurar la confianza entre la aplicación y Kusto:

1. En `Required permissions` la hoja, seleccione `Add`.

    ![texto alternativo](./images/Aad-create-app-step-5.png "Aad-create-app-step-5")

1. Elija `Select an API`, `KustoService` ingrese en el `KustoService (Kusto)`cuadro de filtro y seleccione .
1. Si los clústeres de Kusto requieren MFA, escriba `KustoMFA` en el cuadro de filtro y seleccione `KustoServiceMFA (KustoMFA)`.

    ![texto alternativo](./images/Aad-create-app-step-6.png "Aad-create-app-step-6")

1. Después de confirmar la selección, `Access Kusto`elija permisos delegados para .

    ![texto alternativo](./images/Aad-create-app-step-7.png "Aad-create-app-step-7")

1. Seleccione `Done` esta opción para completar el proceso.



### <a name="2-set-permissions-to-the-application-on-kusto-cluster"></a>2. Establezca los permisos para la aplicación en el clúster de Kusto

> Antes de usar la aplicación recién aprovisionada, compruebe que se resuelve desde Graph API.<br>
    Los cambios de AAD pueden tardar hasta varias horas en propagarse.

1. Acceda al administrador de la base de datos para conceder permisos a la nueva aplicación.
Consulte la sección Administración de [entidades de](../security-roles.md) seguridad de base de datos para obtener más información sobre cómo establecer los permisos.<br>
Para configurar permisos de ingesta, consulte Permisos de [ingesta](../../api/netfx/kusto-ingest-client-permissions.md).
1. Agregue una conexión a este servicio en su Explorador de Kusto, si aún no lo tiene.

### <a name="3-application-can-now-access-kusto"></a>3. La aplicación ahora puede acceder a Kusto

Cuando utilice la aplicación recién aprovisionada para acceder a clústeres de Kusto mediante bibliotecas de cliente de Kusto, especifique la siguiente cadena de conexión (si eligió autenticarse con una clave compartida):

`https://`*ClusterDnsName*`/;Federated Security=True;Application Client Id=`*ApplicationClientId*`;Application Key=`*ApplicationKey*


### <a name="appendix-a-aad-errors"></a>Apéndice A: Errores de AAD

#### <a name="aad-error-aadsts650057"></a>AAD Error AADSTS650057

Si la aplicación se utiliza para autenticar usuarios o aplicaciones para el acceso de Kusto, debe configurar permisos delegados para la aplicación de servicio Kusto, es decir, declarar que la aplicación puede autenticar usuarios o aplicaciones para el acceso de Kusto.
Si no lo hace, se producirá un error similar al siguiente cuando se realice un intento de autenticación:

`AADSTS650057: Invalid resource. The client has requested access to a resource which is not listed in the requested permissions in the client's application registration...`

Deberá seguir las instrucciones para configurar permisos delegados para la aplicación de [servicio Kusto.](#set-up-delegated-permissions-for-kusto-service-application)

#### <a name="enable-user-consent-if-needed"></a>Habilite el consentimiento del usuario si es necesario

El administrador de inquilinos de AAD puede promulgar una directiva que impida a los usuarios de inquilino dar su consentimiento a las aplicaciones. Esto dará lugar a un error similar al siguiente, cuando un usuario intenta iniciar sesión en la aplicación:

`AADSTS65001: The user or administrator has not consented to use the application with ID '<App ID>' named 'App Name'`

Deberá solicitar al administrador de AAD que conceda el consentimiento para todos los usuarios del inquilino o que habilite el consentimiento del usuario para la aplicación específica.



### <a name="appendix-b-advanced-topics-and-troubleshooting"></a>Apéndice B: Temas avanzados y solución de problemas

* Consulte la documentación de cadenas de [conexión de Kusto](../../api/connection-strings/kusto.md) para obtener una referencia completa de las cadenas de conexión admitidas
