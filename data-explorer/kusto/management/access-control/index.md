---
title: 'Introducción al control de acceso de Kusto: Azure Data Explorer | Microsoft Docs'
description: En este artículo se proporciona información general acerca del control de acceso de Kusto en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
ms.date: 11/25/2019
ms.openlocfilehash: ecdcdf22fe25c855045d90e294597c1abc769c03
ms.sourcegitcommit: f6cf88be736aa1e23ca046304a02dee204546b6e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2020
ms.locfileid: "82862112"
---
# <a name="kusto-access-control-overview"></a>Introducción al control de acceso de Kusto

El control de acceso de Kusto se basa en dos dimensiones:
* [Autenticación](#authentication): se valida la identidad de la entidad de seguridad que realiza una solicitud.
* [Autorización](#authorization): se valida que la entidad de seguridad que realiza una solicitud tenga permiso para realizar esa solicitud en el recurso de destino.

Para ejecutar correctamente una consulta o un comando de control en un clúster, una base de datos o una tabla de Kusto, el actor debe superar correctamente tanto la autenticación como la autorización.

## <a name="authentication"></a>Authentication


**Azure Active Directory (AAD)** es el servicio de directorio en la nube multiinquilino preferido de Azure, capaz de autenticar entidades de seguridad o federarse con otros proveedores de identidades, como Microsoft Active Directory.

AAD es el método preferido para autenticarse en Kusto en Microsoft. Admite varios escenarios de autenticación:
* **Autenticación de usuario** (inicio de sesión interactivo): se usa para autenticar entidades de seguridad humanas.
* **Autenticación de aplicación** (inicio de sesión no interactivo): se usa para autenticar aquellos servicios y aplicaciones que tienen que ejecutarse o autenticarse sin que haya usuarios humanos presentes.

### <a name="user-authentication"></a>Autenticación de usuarios
La autenticación de usuarios se produce cuando el usuario presenta las credenciales a AAD (o a algún proveedor de identidades que trabaje con AAD, como ADFS) y recibe un token de seguridad que se puede presentar al servicio Kusto. Al servicio Kusto le da igual cómo se haya obtenido el token de seguridad, lo que le importa es si el token es válido y la información que AAD (o el proveedor de identidades federado) coloca en él.

En el lado del cliente, Kusto admite tanto la autenticación interactiva, en la que la biblioteca cliente de AAD ADAL o un código similar solicitan al usuario que escriba las credenciales. También admite la autenticación basada en token, en la que la aplicación que usa Kusto obtiene un token de usuario válido y lo presenta. Por último, admite un escenario en el que la aplicación que usa Kusto obtiene un token de usuario válido para otro servicio (que no es Kusto), siempre que haya una relación de confianza entre ese recurso y Kusto.

Consulte las [cadenas de conexión de Kusto](../../api/connection-strings/kusto.md) para más información sobre cómo usar las bibliotecas cliente de Kusto y autenticarse con AAD en Kusto.

### <a name="application-authentication"></a>Autenticación de la aplicación
Cuando las solicitudes no están asociadas a ningún usuario concreto o no hay ningún usuario disponible para especificar las credenciales, se puede usar el flujo de autenticación de la aplicación de AAD. En este flujo, la aplicación se autentica en AAD (o en el proveedor de identidades federado) mediante la presentación de cierta información secreta. Los distintos clientes de Kusto admiten los siguientes escenarios:

* Autenticación de la aplicación mediante un certificado X.509v2 instalado localmente.
* Autenticación de la aplicación mediante un certificado X.509v2 proporcionado a la biblioteca cliente como un flujo de bytes.
* Autenticación de la aplicación mediante un identificador de la aplicación de AAD y una clave de la aplicación de AAD (el equivalente de la autenticación con nombre de usuario/contraseña para aplicaciones).
* Autenticación de la aplicación mediante un token de AAD válido obtenido previamente (generado para Kusto).
* Autenticación de la aplicación mediante un token de AAD válido que se ha obtenido previamente y que se ha emitido para otro recurso, siempre que haya una relación de confianza entre ese recurso y Kusto.


### <a name="microsoft-accounts-msas"></a>Cuentas Microsoft (MSA)
Cuentas Microsoft (MSA) es el término que se usa para denominar todas las cuentas de usuario administradas por Microsoft que no pertenecen a ninguna organización `hotmail.com`, `live.com`, `outlook.com`.
Kusto admite la autenticación de usuario en las MSA (tenga en cuenta que no existe el concepto de grupos de seguridad), que se identifican mediante su UPN (nombre de entidad de seguridad universal).
Cuando se configure una entidad de seguridad de MSA en un recurso, Kusto **no** intentará resolver el UPN proporcionado.

### <a name="authenticated-sdk-or-rest-calls"></a>Llamadas a REST o SDK autenticadas
* Cuando se usa la API REST, la autenticación se realiza mediante el encabezado HTTP estándar `Authorization`.
* Cuando se usa cualquiera de las bibliotecas .NET de Kusto, para controlar la autenticación se especifica el método de autenticación y los parámetros en la [cadena de conexión de Kusto](../../api/connection-strings/kusto.md), o bien estableciendo las propiedades en el objeto [Propiedades de la solicitud del cliente](https://kusto.azurewebsites.net/docs/api/request-properties.html).

### <a name="kusto-client-sdk-as-an-aad-client-application"></a>SDK de cliente de Kusto como aplicación cliente de AAD
Cuando las bibliotecas cliente de Kusto invocan a ADAL (la biblioteca cliente de AAD) para adquirir un token para comunicarse con Kusto, proporcionan la siguiente información:

1. El recurso (URI del clúster, por ejemplo `https://Cluster-and-region.kusto.windows.net`)
2. El identificador de la aplicación cliente de AAD
3. El identificador URI de redireccionamiento de la aplicación cliente de AAD
4. El inquilino de AAD (esto afecta al punto de conexión de AAD que se usa para la autenticación; por ejemplo, en el caso del inquilino de AAD `microsoft.com`, el punto de conexión de AAD sería `https://login.microsoftonline.com/microsoft.com`)

El token que devuelve ADAL a la biblioteca cliente de Kusto tiene la dirección URL del clúster de Kusto adecuado como audiencia, y el permiso de "acceso a Kusto" como ámbito.

**Ejemplo: obtención de un token de usuario de AAD para un clúster de Kusto**
```csharp
// Create Auth Context for AAD (common or tenant-specific endpoint):
AuthenticationContext authContext = new AuthenticationContext("https://login.microsoftonline.com/{AAD TenantID or name}");

// Provide your Application ID and redirect URI
var clientAppID = "{your client app id}";
var redirectUri = new Uri("{your client app redirect uri}");

// acquireTokenTask will receive the bearer token for the authenticated user
var acquireTokenTask = authContext.AcquireTokenAsync(
    $"https://{clusterNameAndRegion}.kusto.windows.net",
    clientAppID,
    redirectUri,
    new PlatformParameters(PromptBehavior.Auto, null)).GetAwaiter().GetResult();
```


## <a name="authorization"></a>Authorization

Todas las entidades de seguridad autenticadas, independientemente del método que se haya usado para autenticarlas, también se someten a una comprobación de la autorización antes de que se les permita realizar una acción en cualquier recurso de Kusto.
Kusto usa un [modelo de autorización basado en roles](role-based-authorization.md). Las entidades de seguridad se asignan a uno o varios **roles de seguridad** y la autorización se realiza correctamente siempre que alguno de los roles de la entidad de seguridad esté autorizado.

Por ejemplo, el **rol de usuario de base de datos** concede a las entidades de seguridad (usuarios o servicios) permisos para leer los datos de una base de datos concreta, crear tablas en la base de datos y crear funciones en ella.

La asociación de entidades de seguridad con roles de seguridad se puede definir de forma individual o mediante grupos de seguridad (definidos en AAD). Los comandos individuales para hacerlo se definen en [Establecimiento de reglas de autorización basadas en roles](../security-roles.md).
