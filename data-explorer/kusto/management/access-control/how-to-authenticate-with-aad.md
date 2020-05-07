---
title: 'Autenticación con AAD para Azure Explorador de datos Access: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe cómo autenticarse con AAD para Azure Explorador de datos Access en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
ms.date: 09/13/2019
ms.openlocfilehash: b7e2120f158093e07e096b200b96ac3e265ae2e0
ms.sourcegitcommit: f6cf88be736aa1e23ca046304a02dee204546b6e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2020
ms.locfileid: "82861993"
---
# <a name="how-to-authenticate-with-aad-for-azure-data-explorer-access"></a>Autenticación con AAD para Azure Explorador de datos Access

La manera recomendada de acceder a Azure Explorador de datos es autenticarse en el servicio **Azure Active Directory** (a veces también denominado **Azure ad**o simplemente **AAD**). De este modo, garantiza que Azure Explorador de datos nunca ve las credenciales de directorio de la entidad de seguridad de acceso, mediante un proceso de dos fases:

1. En el primer paso, el cliente se comunica con el servicio AAD, se autentica en él y solicita un token de acceso emitido específicamente para el punto de conexión de Azure Explorador de datos concreto al que el cliente tiene acceso.
2. En el segundo paso, el cliente emite solicitudes a Azure Explorador de datos, lo que proporciona el token de acceso adquirido en el primer paso como una prueba de identidad a Azure Explorador de datos.

Después, Azure Explorador de datos ejecuta la solicitud en nombre de la entidad de seguridad para la que AAD emitió el token de acceso y todas las comprobaciones de autorización se realizan con esta identidad.

En la mayoría de los casos, la recomendación es usar uno de los SDK de Explorador de datos de Azure para tener acceso al servicio mediante programación, ya que eliminan gran parte de las molestias de implementar el flujo anterior (y mucho más). Consulte, por ejemplo, el [SDK de .net](../../api/netfx/about-the-sdk.md).
Después, las propiedades de autenticación se establecen mediante la [cadena de conexión Kusto](../../api/connection-strings/kusto.md).
Si eso no es posible, siga leyendo para obtener información detallada sobre cómo implementar este flujo usted mismo.

Los principales escenarios de autenticación son:

* **Una aplicación cliente que autentica a un usuario con sesión iniciada**.
  En este escenario, una aplicación interactiva (cliente) desencadena un mensaje de AAD para el usuario para las credenciales (como nombre de usuario y contraseña).
  Consulte [autenticación de usuarios](#user-authentication),

* **Una aplicación "sin periféricos"**.
  En este escenario, se ejecuta una aplicación sin ningún usuario presente para proporcionar credenciales y, en su lugar, la aplicación se autentica como "a través de" a AAD mediante el uso de algunas credenciales con las que se ha configurado.
  Consulte [autenticación](#application-authentication)de la aplicación.

* **Autenticación en nombre de**.
  En este escenario, a veces denominado "servicio Web" o "aplicación web", la aplicación obtiene un token de acceso de AAD desde otra aplicación y, a continuación, lo convierte en otro token de acceso de AAD que se puede usar con Azure Explorador de datos.
  En otras palabras, la aplicación actúa como un mediador entre el usuario o la aplicación que proporcionó credenciales y el servicio Azure Explorador de datos.
  Vea la [autenticación en nombre de](#on-behalf-of-authentication).

## <a name="specifying-the-aad-resource-for-azure-data-explorer"></a>Especificación del recurso de AAD para Azure Explorador de datos

Al adquirir un token de acceso desde AAD, el cliente debe indicar a AAD a qué **recurso de AAD** se debe emitir el token. El recurso de AAD de un punto de conexión de Azure Explorador de datos es el URI del punto de conexión, con lo que la información del puerto y la ruta de acceso son. Por ejemplo:

```txt
https://help.kusto.windows.net
```



## <a name="specifying-the-aad-tenant-id"></a>Especificación del identificador de inquilino de AAD

AAD es un servicio multiempresa y cada organización puede crear un objeto denominado **Directory** en AAD. El objeto de directorio contiene objetos relacionados con la seguridad, como cuentas de usuario, aplicaciones y grupos. AAD a menudo hace referencia al directorio como un **inquilino**. Los inquilinos de AAD se identifican mediante un GUID (**identificador de inquilino**). En muchos casos, los inquilinos de AAD también pueden identificarse por el nombre de dominio de la organización.

Por ejemplo, una organización llamada "Contoso" puede tener el identificador `4da81d62-e0a8-4899-adad-4349ca6bfe24` de inquilino y el nombre `contoso.com`de dominio.

## <a name="specifying-the-aad-authority"></a>Especificación de la entidad de AAD

AAD tiene varios puntos de conexión para la autenticación:

* Cuando se conoce el inquilino que hospeda la entidad de seguridad que se está autenticando (es decir, cuando sabe en qué directorio de AAD se encuentra el usuario o la aplicación `https://login.microsoftonline.com/{tenantId}`), el punto de conexión de AAD es.
  Aquí, `{tenantId}` es el identificador de inquilino de la organización en AAD o su nombre de dominio (por `contoso.com`ejemplo,).

* Cuando no se conoce el inquilino que hospeda la entidad de seguridad que se está autenticando, se puede usar el punto de `{tenantId}` conexión "común" `common`reemplazando lo anterior por el valor.

> [!NOTE]
> El punto de conexión de AAD que se usa para la autenticación también se denomina **dirección URL** de la entidad de AAD o simplemente **entidad de AAD**.

## <a name="aad-token-cache"></a>Caché de tokens de AAD

Al usar el SDK de Azure Explorador de datos, los tokens de AAD se almacenan en el equipo local en una caché de tokens por usuario (un archivo denominado **%AppData%\Kusto\tokenCache.Data** al que solo puede acceder o descifrar el usuario que inició sesión). La memoria caché se inspecciona para buscar tokens antes de solicitar las credenciales al usuario, lo que reduce en gran medida el número de veces que un usuario tiene que escribir las credenciales.

> [!NOTE]
> La caché de tokens de AAD reduce el número de mensajes interactivos que se presentarán a un usuario con acceso a Azure Explorador de datos, pero no los reduce. Además, los usuarios no pueden anticiparse de antemano cuando se les solicitarán las credenciales.
> Esto significa que no debe intentar usar una cuenta de usuario para tener acceso a Azure Explorador de datos si es necesario admitir inicios de sesión no interactivos (por ejemplo, cuando se programan tareas, por ejemplo), porque cuando llega el momento a solicitar al usuario que ha iniciado sesión las credenciales, se producirá un error si se ejecuta en un inicio de sesión no interactivo.


## <a name="user-authentication"></a>Autenticación de usuarios

La forma más sencilla de acceder a Azure Explorador de datos con la autenticación de usuario es usar el SDK de Azure `Federated Authentication` explorador de datos y establecer la propiedad de la cadena `true`de conexión de Azure explorador de datos en. La primera vez que se usa el SDK para enviar una solicitud al servicio, se presentará al usuario un formulario de inicio de sesión para especificar las credenciales de AAD y, si la autenticación se realiza correctamente, se enviará la solicitud.

Las aplicaciones que no usan el SDK de Azure Explorador de datos pueden seguir usando la biblioteca de cliente de AAD (ADAL) en lugar de implementar el cliente del Protocolo de seguridad del servicio AAD. Consulte [https://github.com/AzureADSamples/WebApp-WebAPI-OpenIDConnect-DotNet] para obtener un ejemplo de cómo hacerlo desde una aplicación .net.

Para autenticar a los usuarios para el acceso a Azure Explorador de datos, primero se debe `Access Kusto` conceder el permiso delegado a una aplicación. Para más información, consulte la [Guía de Kusto para el aprovisionamiento de aplicaciones de AAD](how-to-provision-aad-app.md#set-up-delegated-permissions-for-kusto-service-application) .

El siguiente fragmento de código breve muestra cómo usar ADAL para adquirir un token de usuario de AAD para acceder a Azure Explorador de datos (inicia la UI de inicio de sesión):

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

El siguiente fragmento de código breve muestra cómo usar ADAL para adquirir un token de aplicación de AAD para acceder a Azure Explorador de datos. En este flujo no se presenta ningún aviso y la aplicación debe registrarse con AAD y estar equipada con las credenciales necesarias para realizar la autenticación de la aplicación (por ejemplo, una clave de aplicación emitida por AAD o un certificado X509v2 que se ha registrado previamente con AAD).

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

En este escenario, se envió a una aplicación un token de acceso de AAD para algún recurso arbitrario administrado por la aplicación y se usa ese token para adquirir un nuevo token de acceso de AAD para el recurso de Explorador de datos de Azure para que la aplicación pueda acceder a Kusto en nombre de la entidad de seguridad indicada por el token de acceso de AAD original.

Este flujo se denomina [flujo de intercambio de tokens de OAuth2](https://tools.ietf.org/html/draft-ietf-oauth-token-exchange-04).
Por lo general, requiere varios pasos de configuración con AAD y, en algunos casos (según la configuración del inquilino de AAD), puede requerir un consentimiento especial por parte del administrador del inquilino de AAD.



**Paso 1: establecer una relación de confianza entre la aplicación y el servicio Azure Explorador de datos**

1. Abra el [Azure portal](https://portal.azure.com/) y asegúrese de que ha iniciado sesión en el inquilino correcto (consulte la esquina superior/derecha de la identidad usada para iniciar sesión en el portal).

2. En el panel recursos, haga clic en **Azure Active Directory**y, a continuación, **registros de aplicaciones**.

3. Busque la aplicación que usa el flujo "en nombre de" y ábrala.

4. Haga clic en **permisos de API**y, a continuación, en **Agregar un permiso**.

5. Busque la aplicación denominada **Azure explorador de datos** y selecciónela.

6. Seleccione **user_impersonation/Access Kusto**.

7. Haga clic en **Agregar permiso**.

**Paso 2: realizar el intercambio de tokens en el código del servidor**

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

**Paso 3: proporcionar el token a la biblioteca de cliente de Kusto y ejecutar consultas**

```csharp
var kcsb = new KustoConnectionStringBuilder(string.Format(
    "https://{0}.kusto.windows.net;fed=true;UserToken={1}",
    clusterName,
    tokenForKusto.AccessToken));
var client = KustoClientFactory.CreateCslQueryProvider(kcsb);
var queryResult = client.ExecuteQuery(databaseName, query, null);
```

## <a name="web-client-javascript-authentication-and-authorization"></a>Autenticación y autorización del cliente web (JavaScript)



**Configuración de la aplicación AAD**

> [!NOTE]
> Además de los [pasos](./how-to-provision-aad-app.md) estándar que debe seguir para configurar una aplicación de AAD, también debe habilitar el flujo implícito de OAuth en su aplicación de AAD. Para ello, seleccione manifiesto en la página de la aplicación en Azure portal y establezca oauth2AllowImplicitFlow en true.

**Detalles**

Cuando el cliente es un código JavaScript que se ejecuta en el explorador del usuario, se usa el flujo de concesión implícito. El token que concede el acceso de la aplicación cliente al servicio Azure Explorador de datos se proporciona inmediatamente después de una autenticación correcta como parte del URI de redirección (en un fragmento de URI); en este flujo no se proporciona ningún token de actualización, por lo que el cliente no puede almacenar en caché el token durante períodos de tiempo prolongados y reutilizarlo.

Al igual que en el flujo de Native Client, debe haber dos aplicaciones de AAD (servidor y cliente) con una relación configurada entre ellas.

AdalJs requiere obtener un id_token antes de que se realicen las llamadas a access_token.

El `AuthenticationContext.login()` token de acceso se obtiene llamando al método y access_tokens se obtienen mediante una llamada `Authenticationcontext.acquireToken()`a.

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

* Llame `authContext.login()` a antes de `acquireToken()` intentar si no ha iniciado sesión. una buena forma de saber si ha iniciado sesión o no es llamar `authContext.getCachedUser()` a y ver si devuelve) `false`
* Llame `authContext.handleWindowCallback()` a cada vez que se cargue la página. Este es el fragmento de código que intercepta el redireccionamiento desde AAD y extrae el token de la dirección URL de fragmento y lo almacena en caché.
* Llame `authContext.acquireToken()` a para obtener el token de acceso real, ahora que tiene un token de identificador válido. El primer parámetro de acquireToken será la dirección URL del recurso de aplicación de AAD del servidor de Kusto.

```javascript
 authContext.acquireToken("<Kusto cluster URL>", callbackThatUsesTheToken);
 ```

* en callbackThatUsesTheToken, puede usar el token como un token de portador en la solicitud de Azure Explorador de datos. Por ejemplo:

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

> ADVERTENCIA: Si obtiene la siguiente excepción o similar a la autenticación:`ReferenceError: AuthenticationContext is not defined`
probablemente, es porque no tiene AuthenticationContext en el espacio de nombres global.
Desafortunadamente, AdalJS actualmente tiene un requisito no documentado que indica que el contexto de autenticación se definirá en el espacio de nombres global.