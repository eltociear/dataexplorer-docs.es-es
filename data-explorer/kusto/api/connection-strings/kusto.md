---
title: 'Cadenas de conexión de Kusto: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las cadenas de conexión de Kusto en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: bf31d8573266de1217ce93944a357c716d3ba508
ms.sourcegitcommit: e1e35431374f2e8b515bbe2a50cd916462741f49
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2020
ms.locfileid: "82108174"
---
# <a name="kusto-connection-strings"></a>Cadenas de conexión de Kusto

Las cadenas de conexión de Kusto pueden proporcionar la información necesaria para que una aplicación cliente de Kusto establezca una conexión con un extremo de servicio de Kusto. Las cadenas de conexión de Kusto se modelan después de las cadenas de conexión de ADO.NET. Es decir, la cadena de conexión es una lista delimitada por signos de punto y coma de pares de parámetros de nombre/valor, con el prefijo de un solo URI.

**Ejemplo:**

```text
https://help.kusto.windows.net/Samples; Fed=true; Accept=true
```

El URI proporciona el punto de conexión de servicio para comunicarse con:

* (`https://help.kusto.windows.net`): valor de la `Data Source` propiedad.
* `Samples`(base de datos predeterminada): valor`Initial Catalog` de la propiedad.

Se proporcionan dos propiedades adicionales mediante la sintaxis de nombre/valor: 

* `Fed`propiedad (también denominada `AAD Federated Security`) establecida en `true`.
* `Accept`propiedad establecida en `true`.

> [!NOTE]
>
> * Los nombres de propiedad no distinguen mayúsculas de minúsculas y los espacios entre los pares de nombre y valor se omiten.
> * **Los valores de propiedad distinguen** mayúsculas de minúsculas. Un valor de propiedad que contiene un punto y`;`coma (), una comilla`'`simple () o una comilla doble`"`() deben ir entre comillas dobles.

Varias herramientas de cliente de Kusto admiten una extensión sobre el prefijo de URI de la cadena de conexión, ya `@` que permiten usar el formato abreviado _ClusterName_ `/` _InitialCatalog_ .
Por `@help/Samples` ejemplo, estas herramientas traducen la cadena de conexión a `https://help.kusto.windows.net/Samples; Fed=true`, que indica tres propiedades (`Data Source`, `Initial Catalog`y `AAD Federated Security`).

Mediante programación, la clase de C# `Kusto.Data.KustoConnectionStringBuilder` puede analizar y manipular las cadenas de conexión de Kusto. Esta clase valida todas las cadenas de conexión y genera una excepción en tiempo de ejecución si se produce un error en la validación.
Esta funcionalidad está presente en todos los tipos de SDK de Kusto.

## <a name="connection-string-properties"></a>Propiedades de cadena de conexión

En la tabla siguiente se enumeran todas las propiedades que se pueden especificar en una cadena de conexión de Kusto.
Muestra los nombres de programación (que es el nombre de la propiedad del `Kusto.Data.KustoConnectionStringBuilder` objeto), así como los nombres de propiedades adicionales que son alias.

### <a name="general-properties"></a>Propiedades generales

| Nombre de propiedad              | Nombres alternativos                      | Nombre de programación  | Descripción                                                                                                                          |
|----------------------------|----------------------------------------|--------------------|---------------------------------------------------|
| Versión de cliente para seguimiento |                                        | TraceClientVersion | Cuando realice un seguimiento de la versión del cliente, use este valor   |
| Origen de datos                | Dirección, dirección, dirección de red, servidor | DataSource         | El URI que especifica el extremo de servicio Kusto. Por ejemplo, `https://mycluster.kusto.windows.net` o `net.tcp://localhost`.               |
| Catálogo original            | Base de datos                               | InitialCatalog     | Nombre de la base de datos que se va a usar de forma predeterminada. Por ejemplo, mi base de datos|
| Coherencia de consultas          | QueryConsistency                       | QueryConsistency   | Se establece en `strongconsistency` o `weakconsistency` para determinar si la consulta se debe sincronizar con los metadatos antes de ejecutarse. |

### <a name="user-authentication-properties"></a>Propiedades de autenticación de usuario

| Nombre de propiedad          | Nombres alternativos                          | Nombre de programación | Descripción                       |
|------------------------|--------------------------------------------|-------------------|-----------------------------------|
| Seguridad federada de AAD | Seguridad federada, federada, FED, AADFed | FederatedSecurity | Un valor booleano que indica al cliente que realice Azure Active  |
| Exigir MFA            | MFA, EnforceMFA                             | EnforceMfa        | Un valor booleano que indica al cliente que adquiera un token de autenticación multifactor.       |
| Id. de usuario                | UID, usuario                                  | UserID            | Valor de cadena que indica al cliente que realice la autenticación de usuario con el nombre de usuario indicado           |
| Nombre de usuario para el seguimiento  |                                            | TraceUserName     | Valor de cadena que notifica al servicio el nombre de usuario que se va a utilizar al hacer un seguimiento de la solicitud internamente.         |
| Token de usuario             | UsrToken, UserToken                        | UserToken         | Valor de cadena que indica al cliente que realice la autenticación de usuario con el token de portador especificado.<br/>Invalida ApplicationClientId, ApplicationKey y ApplicationToken. (Si se especifica, omite el flujo de autenticación del cliente real en favor del token proporcionado).       |
| Espacio de nombres              | NS                                         | Espacio de nombres         | (Para un uso futuro)  |



### <a name="application-authentication-properties"></a>Propiedades de autenticación de la aplicación

|Nombre de propiedad                                     |Nombres alternativos                         |Nombre de programación                             |Descripción      |
|--------------------------------------------------|------------------------------------------|----------------------------------------------|-----------------|
|Seguridad federada de AAD                            |Seguridad federada, federada, FED, AADFed|FederatedSecurity                             |Un valor booleano que indica al cliente que realice la autenticación federada de Azure Active Directory (AAD).|
|Huella digital del certificado de aplicación                |AppCert                                   |ApplicationCertificateThumbprint              |Valor de cadena que proporciona la huella digital del certificado de cliente que se va a usar al usar un flujo de autenticación de certificado de cliente de aplicación.|
|Identificador de cliente de aplicación                             |AppClientId                               |ApplicationClientId                           |Valor de cadena que proporciona el identificador de cliente de la aplicación que se va a usar al autenticarse|
|Clave de aplicación                                   |AppKey                                    |ApplicationKey                                |Valor de cadena que proporciona la clave de aplicación que se va a utilizar al autenticar mediante un flujo de secreto de la aplicación.|
|Nombre de aplicación para seguimiento                      |TraceAppName                              |ApplicationNameForTracing                     |Valor de cadena que notifica al servicio el nombre de aplicación que se va a utilizar al hacer el seguimiento de la solicitud internamente.|
|Token de aplicación                                 |AppToken                                  |ApplicationToken                              |Valor de cadena que indica al cliente que realice la autenticación de la aplicación con el token de portador especificado.|
|Identificador de autoridad                                      |TenantId                                  |Autoridad                                     |Valor de cadena que proporciona el nombre o el identificador del inquilino en el que se registra la aplicación.|
|                                                  |                                          |EmbeddedManagedIdentity                       |Valor de cadena que indica al cliente qué identidad de aplicación se va a usar con la autenticación de identidad administrada. se `system` usa para indicar la identidad asignada por el sistema. Esta propiedad no se puede establecer con una cadena de conexión, solo mediante programación.|ManagedServiceIdentity                        |TAREA PENDIENTE|
|Nombre distintivo del sujeto del certificado de aplicación|Asunto del certificado de aplicación           |ApplicationCertificateSubjectDistinguishedName||
|Nombre distintivo del emisor del certificado de aplicación |Emisor de certificado de aplicación            |ApplicationCertificateIssuerDistinguishedName ||
|Certificado público de envío de certificado de aplicación   |Certificado de aplicación SendX5c, SendX5c  |ApplicationCertificateSendPublicCertificate   ||



### <a name="client-communication-properties"></a>Propiedades de comunicación de cliente

|Nombre de propiedad                      |Nombres alternativos|Nombre de programación  |Descripción                                                   |
|-----------------------------------|-----------------|-------------------|--------------------------------------------------------------|
|Accept      ||Accept      |Valor booleano que solicita que se devuelvan objetos de error detallados en caso de error.|
|Streaming   ||Streaming   |Un valor booleano que solicita que el cliente no acumule los datos antes de proporcionarlos al llamador.|
|Sin comprimir||Sin comprimir|Un valor booleano que solicita al cliente no le pedirá la compresión de nivel de transporte.|

## <a name="authentication-properties-details"></a>Propiedades de autenticación (detalles)

Una de las tareas importantes de la cadena de conexión es indicar al cliente cómo autenticarse en el servicio.
Los clientes suelen usar el siguiente algoritmo para la autenticación con puntos de conexión HTTP/HTTPS:

1. Si AadFederatedSecurity es true:
    1. Si se especifica UserToken, use la autenticación federada de AAD con el token especificado.
    1. De lo contrario, si se especifica ApplicationToken, realice la autenticación federada con el token especificado.
    1. De lo contrario, si se especifican ApplicationClientId y ApplicationKey, realice la autenticación federada con la clave y el identificador de cliente de la aplicación especificados.
    1. De lo contrario, si se especifican ApplicationClientId y ApplicationCertificateThumbprint, realice la autenticación federada con el identificador y el certificado de cliente de la aplicación especificados.
    1. De lo contrario, realice la autenticación federada con la identidad del usuario que ha iniciado la sesión actual (se le pedirá al usuario si se trata de la primera autenticación de la sesión).

1. De lo contrario, no se autentica.





### <a name="aad-federated-application-authentication-with-application-certificate"></a>Autenticación de aplicaciones federadas de AAD con certificado de aplicación

1. La autenticación basada en el certificado de una aplicación solo se admite para las aplicaciones web (y no para las aplicaciones cliente nativas).
1. La aplicación Web debe estar configurada para aceptar el certificado especificado. [Autenticación basada en el certificado de la aplicación AAD](https://github.com/Azure-Samples/active-directory-dotnet-daemon-certificate-credential)
1. La aplicación Web debe configurarse como entidad de seguridad autorizada en el clúster de Kusto pertinente.
1. El certificado con la huella digital determinada debe estar instalado (en el almacén de la máquina local o en el almacén del usuario actual).
1. La clave pública del certificado debe contener al menos 2048 bits.

## <a name="aad-based-authentication-examples"></a>Ejemplos de autenticación basada en AAD

**Autenticación federada de AAD basada en la identidad del usuario que ha iniciado la sesión actual (se le pedirá al usuario si es necesario)**

```csharp
// Option 1
var serviceName = "help";
var authority = "contoso.com"; // Or the AAD tenant GUID: "..."
var kustoConnectionStringBuilder = new KustoConnectionStringBuilder($"https://{serviceNameAndRegion}.kusto.windows.net")
  .WithAadUserPromptAuthentication(authority);

// Option 2
var serviceName = "help";
var authority = "contoso.com"; // Or the AAD tenant GUID: "..."
var kustoConnectionStringBuilder = new KustoConnectionStringBuilder($"https://{serviceNameAndRegion}.kusto.windows.net")
{
    FederatedSecurity = true,
    InitialCatalog = "NetDefaultDB",
    Authority = authority,
};

// Equivalent Kusto connection string: $"Data Source=https://{serviceNameAndRegion}.kusto.windows.net:443;Database=NetDefaultDB;Fed=True;authority={authority}"
```

**Autenticación de aplicaciones federadas de AAD basada en un ApplicationClientId y ApplicationKey determinados**

```csharp
// Option 1
var serviceName = "help";
var authority = "contoso.com"; // Or the AAD tenant GUID: "..."
var applicationClientId = APP_GUID;
var applicationKey = secret;
var kustoConnectionStringBuilder = new KustoConnectionStringBuilder($"https://{serviceNameAndRegion}.kusto.windows.net")
    .WithAadApplicationKeyAuthentication(applicationClientId, applicationKey, authority);

// Option 2
var serviceName = "help";
var authority = "contoso.com"; // Or the AAD tenant GUID: "..."
var applicationClientId = <ApplicationClientId>;
var applicationKey = <ApplicationKey>;
var kustoConnectionStringBuilder = new KustoConnectionStringBuilder(@"https://{serviceNameAndRegion}.kusto.windows.net")
{
    FederatedSecurity = true,
    InitialCatalog = "NetDefaultDB",
    ApplicationClientId = applicationClientId,
    ApplicationKey = applicationKey,
    Authority = authority,
};

// Equivalent Kusto connection string: $"Data Source=https://{serviceNameAndRegion}.kusto.windows.net:443;Database=NetDefaultDB;Fed=True;AppClientId={applicationClientId};AppKey={applicationKey};authority={authority}"
```

**Autenticación federada de AAD basada en el token de la aplicación o del usuario determinado**

```csharp
var serviceNameAndRegion = "help";
var databaseName = "NetDefaultDB";
var clusterAndDatabase = string.Format(
    "https://{0}.kusto.windows.net/{1}",
    serviceNameAndRegion, databaseName);

// AAD User - Option 1
var userToken = "<UserToken>";
var kustoConnectionStringBuilder = new KustoConnectionStringBuilder(clusterAndDatabase)
    .WithAadUserTokenAuthentication(userToken);

// AAD User - Option 2
var userToken = "<UserToken>";
var authority = "contoso.com"; // Or the AAD tenant GUID: "..."
var kustoConnectionStringBuilder = new KustoConnectionStringBuilder(clusterAndDatabase)
{
    FederatedSecurity = true,
    UserToken = userToken,
    Authority = authority,
};

// Equivalent Kusto connection string: "Data Source=https://{serviceNameAndRegion}.kusto.windows.net:443;Database=NetDefaultDB;Fed=True;UserToken={user_token};authority={authority}"

// AAD Application - Option 1
var applicationToken = "<ApplicationToken>";
var kustoConnectionStringBuilder = new KustoConnectionStringBuilder(clusterAndDatabase)
    .WithAadApplicationTokenAuthentication();

// AAD Application - Option 2
var authority = "contoso.com"; // Or the AAD tenant GUID: "..."
var applicationToken = "<UserToken>";
var kustoConnectionStringBuilder = new KustoConnectionStringBuilder(clusterAndDatabase)
{
    FederatedSecurity = true,
    ApplicationToken = applicationToken,
    Authority = authority,
};

// Equivalent Kusto connection string: $"Data Source=https://{serviceNameAndRegion}.kusto.windows.net:443;Database=NetDefaultDB;Fed=True;AppToken={applicationToken};authority={authority}"
```

**Usar huella digital de certificado (el cliente intentará cargar el certificado desde el almacén local)**

```csharp
var serviceNameAndRegion = "help";
var databaseName = "Samples";
var clusterAndDatabase = string.Format(
    "https://{0}.kusto.windows.net/{1}",
    serviceNameAndRegion, databaseName);

string applicationClientId = "<applicationClientId>";
string applicationCertificateThumbprint = "<ApplicationCertificateThumbprint>";

// Option 1
var serviceName = "help";
var authority = "contoso.com"; // Or the AAD tenant GUID: "..."
var kustoConnectionStringBuilder = new KustoConnectionStringBuilder(clusterAndDatabase)
    .WithAadApplicationThumbprintAuthentication(applicationClientId, applicationCertificateThumbprint, authority);

// Option 2
var authority = "contoso.com"; // Or the AAD tenant GUID: "..."
var kustoConnectionStringBuilder = new KustoConnectionStringBuilder(clusterAndDatabase)
{
    FederatedSecurity = true,
    ApplicationClientId = applicationClientId,
    ApplicationCertificateThumbprint = applicationCertificateThumbprint,
    Authority = authority,
};

// Equivalent Kusto connection string: $"Data Source=https://{serviceNameAndRegion}.kusto.windows.net:443;Database=NetDefaultDB;Fed=True;AppClientId={applicationClientId};AppCert={applicationCertificateThumbprint};authority={authority}"
```

