---
title: Cómo autenticarse con AAD para Azure Data Explorer Access - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe Cómo autenticarse con AAD para Azure Data Explorer Access en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/13/2019
ms.openlocfilehash: dc3a87a290ac535ac475af5017170a0478cd9b54
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522923"
---
# <a name="how-to-authenticate-with-aad-for-azure-data-explorer-access"></a>Cómo autenticarse con AAD para Azure Data Explorer Access

La forma recomendada de tener acceso a Azure Data Explorer es autenticarse en el servicio **Azure Active Directory** (a veces también denominado Azure **AD**o simplemente **AAD).** Al hacerlo, garantiza que Azure Data Explorer nunca vea las credenciales de directorio de la entidad de acceso mediante un proceso de dos etapas:

1. En el primer paso, el cliente se comunica con el servicio AAD, se autentica en él y solicita un token de acceso emitido específicamente para el punto de conexión de Azure Data Explorer concreto al que el cliente tiene la intención de tener acceso.
2. En el segundo paso, el cliente emite solicitudes a Azure Data Explorer, proporcionando el token de acceso adquirido en el primer paso como prueba de identidad para Azure Data Explorer.

A continuación, Azure Data Explorer ejecuta la solicitud en nombre de la entidad de seguridad para la que AAD emitió el token de acceso y todas las comprobaciones de autorización se realizan con esta identidad.

En la mayoría de los casos, la recomendación es usar uno de los SDK de Azure Data Explorer para tener acceso al servicio mediante programación, ya que eliminan gran parte de la molestia de implementar el flujo anterior (y mucho más). Vea, por ejemplo, el SDK de [.NET](../../api/netfx/about-the-sdk.md).
A continuación, la cadena de [conexión Kusto](../../api/connection-strings/kusto.md)establece las propiedades de autenticación.
Si eso no es posible, por favor siga leyendo para obtener información detallada sobre cómo implementar este flujo usted mismo.

Los principales escenarios de autenticación son:

* **Una aplicación cliente que autentica un usuario que ha iniciado sesión.**
  En este escenario, una aplicación interactiva (cliente) desencadena un mensaje de AAD al usuario para las credenciales (como nombre de usuario y contraseña).
  Consulte [autenticación de usuario](#user-authentication),

* **Una aplicación "sin cabeza".**
  En este escenario, una aplicación se ejecuta sin ningún usuario presente para proporcionar credenciales y, en su lugar, la aplicación se autentica como "sí mismo" en AAD con algunas credenciales con las que se ha configurado.
  Consulte [Autenticación de aplicaciones](#application-authentication).

* **Autenticación en nombre de la autenticación**.
  En este escenario, a veces denominado escenario de "servicio web" o "aplicación web", la aplicación obtiene un token de acceso de AAD de otra aplicación y, a continuación, lo "convierte" en otro token de acceso de AAD que se puede usar con Azure Data Explorer.
  En otras palabras, la aplicación actúa como mediador entre el usuario o la aplicación que proporcionaba credenciales y el servicio Explorador de datos de Azure.
  Consulte [autenticación en nombre de](#on-behalf-of-authentication).

## <a name="specifying-the-aad-resource-for-azure-data-explorer"></a>Especificación del recurso de AAD para Azure Data Explorer

Al adquirir un token de acceso de AAD, el cliente debe indicar a AAD a qué recurso de **AAD** se debe emitir el token. El recurso de AAD de un punto de conexión de Azure Data Explorer es el URI del punto de conexión, que se puede que la información del puerto y la ruta de acceso. Por ejemplo:

```txt
https://help.kusto.windows.net
```



## <a name="specifying-the-aad-tenant-id"></a>Especificar el identificador de inquilino de AAD

AAD es un servicio multiinquilino y cada organización puede crear un objeto denominado **directorio** en AAD. El objeto de directorio contiene objetos relacionados con la seguridad, como cuentas de usuario, aplicaciones y grupos. AAD a menudo se refiere al directorio como un **inquilino.** Los inquilinos de AAD se identifican mediante un GUID ( identificador de**inquilino**). En muchos casos, los inquilinos de AAD también se pueden identificar por el nombre de dominio de la organización.

Por ejemplo, una organización denominada "Contoso" podría tener el identificador `4da81d62-e0a8-4899-adad-4349ca6bfe24` de inquilino y el nombre `contoso.com`de dominio.

## <a name="specifying-the-aad-authority"></a>Especificar la autoridad de AAD

AAD tiene una serie de puntos finales para la autenticación:

* Cuando se conoce el inquilino que hospeda la entidad de seguridad que se está autenticando (en otras `https://login.microsoftonline.com/{tenantId}`palabras, cuando se sabe en qué directorio de AAD se encuentra el usuario o la aplicación), el extremo de AAD es .
  Aquí, `{tenantId}` es el identificador de inquilino de la organización en AAD o su nombre de dominio (por ejemplo, `contoso.com`).

* Cuando no se conoce el inquilino que hospeda la entidad de seguridad que `{tenantId}` se está `common`autenticando, el punto de conexión "común" se puede usar reemplazando el valor anterior.

> [!NOTE]
> El punto de conexión de AAD utilizado para la autenticación también se denomina dirección URL de autoridad de **AAD** o simplemente autoridad de **AAD.**

## <a name="aad-token-cache"></a>Caché de tokens de AAD

Cuando se usa el SDK de Azure Data Explorer, los tokens de AAD se almacenan en el equipo local en una memoria caché de tokens por usuario (un archivo denominado **%APPDATA%-Kusto-tokenCache.data al** que solo puede tener acceso o descifrar el usuario que ha iniciado sesión.) La memoria caché se inspecciona en busca de tokens antes de solicitar al usuario las credenciales, lo que reduce considerablemente el número de veces que un usuario tiene que escribir las credenciales.

> [!NOTE]
> La caché de tokens de AAD reduce el número de solicitudes interactivas que se presentaría a un usuario con el acceso a Azure Data Explorer, pero no los reduce completos. Además, los usuarios no pueden anticipar de antemano cuándo se les pedirán las credenciales.
> Esto significa que no se debe intentar usar una cuenta de usuario para tener acceso a Azure Data Explorer si es necesario admitir inicios de sesión no interactivos (por ejemplo, al programar tareas, por ejemplo), porque cuando llega el momento de solicitar al usuario que ha iniciado sesión las credenciales que se producirá un error si se ejecuta en el inicio de sesión no interactivo.


## <a name="user-authentication"></a>Autenticación de usuarios

La forma más sencilla de obtener acceso a Azure Data Explorer con `Federated Authentication` autenticación de usuario es `true`usar el SDK de Azure Data Explorer y establecer la propiedad de la cadena de conexión de Azure Data Explorer en . La primera vez que se usa el SDK para enviar una solicitud al servicio, se presentará al usuario un formulario de inicio de sesión para escribir las credenciales de AAD y, en la autenticación correcta, se enviará la solicitud.

Las aplicaciones que no usan el SDK de Azure Data Explorer todavía pueden usar la biblioteca de cliente de AAD (ADAL) en lugar de implementar el cliente de protocolo de seguridad de servicio de AAD. Consulte [https://github.com/AzureADSamples/WebApp-WebAPI-OpenIDConnect-DotNet] para obtener un ejemplo de cómo hacerlo desde una aplicación .NET.

Para autenticar a los usuarios para el acceso `Access Kusto` de Azure Data Explorer, primero se debe conceder el permiso delegado a una aplicación. Consulte la [guía de Kusto sobre](how-to-provision-aad-app.md#set-up-delegated-permissions-for-kusto-service-application) el aprovisionamiento de aplicaciones de AAD para obtener más información.

En el siguiente fragmento de código breve se muestra cómo usar ADAL para adquirir un token de usuario de AAD para tener acceso a Azure Data Explorer (inicia la interfaz de usuario de inicio de sesión):

```csharp
// Create an HTTP request
WebRequest request = WebRequest.Create(new Uri("https://{serviceNameAndRegion}.kusto.windows.net"));

// Create Auth Context for AAD (common or tenant-specific endpoint):
AuthenticationContext authContext = new AuthenticationContext("AAD Authority URL");

// Acquire user token for the interactive user for Kusto:
AuthenticationResult result = authContext.AcquireTokenAsync("https://{serviceNameAndRegion}.kusto.windows.net", "your client app id", 
    new Uri("your client app resource id"), new PlatformParameters(PromptBehavior.Auto)).GetAwaiter().GetResult();

// Extract Bearer access token and set the Authorization header on your request:
string bearerToken = result.AccessToken;
request.Headers.Set(HttpRequestHeader.Authorization, string.Format(CultureInfo.InvariantCulture, "{0} {1}", "Bearer", bearerToken));
```

## <a name="application-authentication"></a>Autenticación de la aplicación

En el siguiente breve fragmento de código se muestra cómo usar ADAL para adquirir un token de aplicación de AAD para tener acceso a Azure Data Explorer. En este flujo no se presenta ningún mensaje, y la aplicación debe estar registrada con AAD y equipada con las credenciales necesarias para realizar la autenticación de la aplicación (como una clave de aplicación emitida por AAD o un certificado X509v2 que se ha registrado previamente con AAD).

```csharp
// Create an HTTP request
WebRequest request = WebRequest.Create(new Uri("https://{serviceNameAndRegion}.kusto.windows.net"));

// Create Auth Context for AAD (common or tenant-specific endpoint):
AuthenticationContext authContext = new AuthenticationContext("AAD Authority URL");

// Acquire application token for Kusto:
ClientCredential applicationCredentials = new ClientCredential("your application client ID", "your application key");
AuthenticationResult result =
        authContext.AcquireTokenAsync("https://{serviceNameAndRegion}.kusto.windows.net", applicationCredentials).GetAwaiter().GetResult();

// Extract Bearer access token and set the Authorization header on your request:
string bearerToken = result.AccessToken;
request.Headers.Set(HttpRequestHeader.Authorization, string.Format(CultureInfo.InvariantCulture, "{0} {1}", "Bearer", bearerToken));
```

## <a name="on-behalf-of-authentication"></a>Autenticación en nombre de

En este escenario, se envió a una aplicación un token de acceso de AAD para algún recurso arbitrario administrado por la aplicación y usa ese token para adquirir un nuevo token de acceso de AAD para el recurso de Azure Data Explorer para que la aplicación pudiera tener acceso a Kusto en nombre de la entidad de seguridad indicada por el token de acceso de AAD original.

Este flujo se denomina flujo de intercambio de [tokens OAuth2.](https://tools.ietf.org/html/draft-ietf-oauth-token-exchange-04)
Por lo general, requiere varios pasos de configuración con AAD y, en algunos casos (dependiendo de la configuración del inquilino de AAD) puede requerir el consentimiento especial del administrador del inquilino de AAD.



**Paso 1: Establezca una relación de confianza entre la aplicación y el servicio Azure Data Explorer**

1. Abra [Azure Portal](https://portal.azure.com/) y asegúrese de que ha iniciado sesión en el inquilino correcto (consulte la esquina superior/derecha de la identidad utilizada para iniciar sesión en el portal).

2. En el panel de recursos, haga clic en **Azure Active Directory**y, a continuación, en **Registros**de aplicaciones .

3. Busque la aplicación que utiliza el flujo en nombre de y ábralo.

4. Haga clic en **Permisos de API y,** a continuación, en **Agregar un permiso**.

5. Busque la aplicación denominada Explorador de datos de **Azure** y selecciónela.

6. Seleccione **user_impersonation / Acceder a Kusto**.

7. Haga clic en **Agregar permiso**.

**Paso 2: Realice el intercambio de tokens en el código del servidor**

```csharp
// Create Auth Context for AAD (common or tenant-specific endpoint):
AuthenticationContext authContext = new AuthenticationContext("AAD Authority URL");

// Exchange your token for for Kusto token.
// You will need to provide your application's client ID and secret to authenticate your application 
var tokenForKusto = authContext.AcquireTokenAsync(
    "https://{serviceNameAndRegion}.kusto.windows.net",
    new ClientCredential(customerAadWebApplicationClientId, customerAAdWebApplicationSecret),
    new UserAssertion(customerAadWebApplicationToken)).GetAwaiter().GetResult();
```

**Paso 3: Proporcione el token a la biblioteca de cliente de Kusto y ejecute consultas**

```csharp
var kcsb = new KustoConnectionStringBuilder(string.Format(
    "https://{0}.kusto.windows.net;fed=true;UserToken={1}", 
    clusterName, 
    tokenForKusto.AccessToken));
var client = KustoClientFactory.CreateCslQueryProvider(kcsb);
var queryResult = client.ExecuteQuery(databaseName, query, null);
```

## <a name="web-client-javascript-authentication-and-authorization"></a>Autenticación y autorización de web Client (JavaScript)



**Configuración de la aplicación AAD**

> [!NOTE]
> Además de los [pasos](./how-to-provision-aad-app.md) estándar que debe seguir para configurar una aplicación de AAD, también debe habilitar el flujo implícito de oauth en la aplicación de AAD. Puede lograrlo seleccionando el manifiesto de la página de la aplicación en el portal azul y estableciendo oauth2AllowImplicitFlow en true.

**Detalles**

Cuando el cliente es un código JavaScript que se ejecuta en el explorador del usuario, se utiliza el flujo de concesión implícito. El token que concede a la aplicación cliente acceso al servicio Explorador de datos de Azure se proporciona inmediatamente después de una autenticación correcta como parte del URI de redirección (en un fragmento de URI); no se proporciona ningún token de actualización en este flujo, por lo que el cliente no puede almacenar en caché el token durante períodos prolongados de tiempo y reutilizarlo.

Al igual que en el flujo de cliente nativo, debe haber dos aplicaciones AAD (servidor y cliente) con una relación configurada entre ellas. 

AdalJs requiere obtener una id_token antes de que se realicen access_token llamadas.

El token de acceso `AuthenticationContext.login()` se obtiene llamando al método `Authenticationcontext.acquireToken()`y access_tokens se obtienen llamando a .

* Cree un AuthenticationContext con la configuración correcta:

```javascript
var config = {
    tenant: "microsoft.com",
    clientId: "<Web AAD app with current website as reply URL. for example, KusDash uses f86897ce-895e-44d3-91a6-aaeae54e278c>",
    redirectUri: "<where you'd like AAD to redirect after authentication succeeds (or fails).>",
    postLogoutRedirectUri: "where you'd like AAD to redirect the browser after logout."
};

var authContext = new AuthenticationContext(config);
```

* Llame `authContext.login()` antes `acquireToken()` de intentar si no ha iniciado sesión. una buena manera de saber si ha iniciado sesión o no es llamar `authContext.getCachedUser()` y ver si devuelve `false`)
* Llame `authContext.handleWindowCallback()` cada vez que se cargue la página. Este es el fragmento de código que intercepta la redirección de AAD y extrae el token de la dirección URL del fragmento y lo almacena en caché.
* Llame `authContext.acquireToken()` para obtener el token de acceso real, ahora que tiene un token de identificador válido. El primer parámetro para acquireToken será la dirección URL del recurso de aplicación de AAD del servidor Kusto.  

```javascript
 authContext.acquireToken("<Kusto cluster URL>", callbackThatUsesTheToken);
 ```

* en callbackThatUsesTheToken puede usar el token como token de portador en la solicitud de Azure Data Explorer. Por ejemplo:

```javascript
var settings = {
    url: "https://" + clusterAndRegionName + ".kusto.windows.net/v1/rest/query",
    type: "POST",
    data: JSON.stringify({
        "db": dbName,
        "csl": query,
        "properties": null
    }),
    contentType: "application/json; charset=utf-8",
    headers: { "Authorization": "Bearer " + token },
    dataType: "json",
    jsonp: false,
    success: function(data, textStatus, jqXHR) {
        if (successCallback !== undefined) {
            successCallback(data.Tables[0]);
        }

    },
    error: function(jqXHR, textStatus, errorThrown) {
        if (failureCallback !==  undefined) {
            failureCallback(textStatus, errorThrown, jqXHR.responseText);
        }

    },
};

$.ajax(settings).then(function(data) {/* do something wil the data */});
``` 

> Advertencia: si obtiene la siguiente excepción o similar al autenticar:`ReferenceError: AuthenticationContext is not defined` 
probablemente se deba a que no tiene AuthenticationContext en el espacio de nombres global. Desafortunadamente, AdalJS actualmente tiene un requisito indocumentado de que el contexto de autenticación se definirá en el espacio de nombres global.