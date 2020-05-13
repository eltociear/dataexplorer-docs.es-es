---
title: 'Solicitud HTTP de consulta/administración: Azure Explorador de datos'
description: En este artículo se describe la solicitud HTTP de consulta/administración en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/27/2020
ms.openlocfilehash: 2c6efc03ea252eba5ed63e99d9214e59113856e9
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373579"
---
# <a name="querymanagement-http-request"></a>Solicitud HTTP de consulta/administración

## <a name="request-verb-and-resource"></a>Recurso y verbo de solicitud

|Acción    |Verbo HTTP|Recurso HTTP   |
|----------|---------|----------------|
|Consultar     |GET      |`/v1/rest/query`|
|Consultar     |POST     |`/v1/rest/query`|
|Consulta V2  |GET      |`/v2/rest/query`|
|Consulta V2  |POST     |`/v2/rest/query`|
|Administración|POST     |`/v1/rest/mgmt` |

Por ejemplo, para enviar un comando de control ("administración") a un punto de conexión de servicio, use la siguiente línea de solicitud:

```
POST https://help.kusto.windows.net/v1/rest/mgmt HTTP/1.1
```

A continuación se muestran los encabezados y el cuerpo de la solicitud que se van a incluir.

## <a name="request-headers"></a>Encabezados de solicitud

La tabla siguiente contiene los encabezados comunes utilizados para las operaciones de consulta y administración.

|Encabezado estándar  |Descripción                                                                                 |Obligatorio/opcional |
|-----------------|--------------------------------------------------------------------------------------------|------------------|
|`Accept`         |Establézcala en `application/json`                                                                   |Obligatorio          |
|`Accept-Encoding`|Las codificaciones admitidas son `gzip` y`deflate`                                                |Opcional          |
|`Authorization`  |Consulte [autenticación](./authentication.md)                                                   |Obligatorio          |
|`Connection`     |Se recomienda habilitar`Keep-Alive`                                                   |Opcional          |
|`Content-Length` |Se recomienda especificar la longitud del cuerpo de la solicitud cuando se conozca                            |Opcional          |
|`Content-Type`   |Establecer en `application/json` con`charset=utf-8`                                              |Obligatorio          |
|`Expect`         |Se puede establecer en`100-Continue`                                                                |Opcional          |
|`Host`           |Establezca en el nombre de dominio completo al que se envió la solicitud (por ejemplo, `help.kusto.windows.net` ). |Obligatorio|

La tabla siguiente contiene los encabezados personalizados comunes que se utilizan para las operaciones de consulta y administración. A menos que se indique lo contrario, estos encabezados se usan solo con fines de telemetría y no tienen ningún impacto en la funcionalidad.

Todos los encabezados son opcionales. Se recomienda especificar el `x-ms-client-request-id` encabezado personalizado. En algunos escenarios, como la cancelación de una consulta en ejecución, este encabezado es obligatorio porque se usa para identificar la solicitud.

|Encabezado personalizado           |Descripción                                                                                               |
|------------------------|----------------------------------------------------------------------------------------------------------|
|`x-ms-app`              |El nombre descriptivo de la aplicación que realiza la solicitud                                                 |
|`x-ms-user`             |El nombre (descriptivo) del usuario que realiza la solicitud                                                        |
|`x-ms-user-id`          |Igual que `x-ms-user`.                                                                                       |
|`x-ms-client-request-id`|Un identificador único para la solicitud.                                                                       |
|`x-ms-client-version`   |El identificador de versión (descriptivo) para el cliente que realiza la solicitud                                       |
|`x-ms-readonly`         |Si se especifica, obliga a que la solicitud se ejecute en modo de solo lectura, lo que impide realizar cambios duraderos |

## <a name="request-parameters"></a>Parámetros de solicitud

Los parámetros siguientes se pueden pasar en la solicitud. Están codificados en la solicitud como parámetros de consulta o como parte del cuerpo, dependiendo de si se usa GET o POST.

|Parámetro   |Descripción                                                                                 |Obligatorio/opcional |
|------------|--------------------------------------------------------------------------------------------|------------------|
|`csl`       |Texto del comando de consulta o de control que se va a ejecutar                                             |Obligatorio          |
|`db`        |Nombre de la base de datos en el ámbito que es el destino del comando de consulta o control            |Opcional para algunos comandos de control. <br>Necesario para otros comandos y todas las consultas. </br>                                                                   |
|`properties`|Proporciona propiedades de solicitud de cliente que modifican el modo en que se procesa la solicitud y sus resultados. Para obtener más información, consulte propiedades de la [solicitud de cliente](../netfx/request-properties.md) .                                               | Opcional         |

## <a name="get-query-parameters"></a>OBTENER parámetros de consulta

Cuando se usa GET, los parámetros de consulta de la solicitud especifican los parámetros de la solicitud.

## <a name="body"></a>Body

Cuando se usa POST, el cuerpo de la solicitud es un único documento JSON codificado en UTF-8, con los valores de los parámetros de la solicitud.

## <a name="examples"></a>Ejemplos

En este ejemplo se muestra la solicitud HTTP POST para una consulta.

```txt
POST https://help.kusto.windows.net/v2/rest/query HTTP/1.1
```

Encabezados de solicitud

```txt
Accept: application/json
Authorization: Bearer ...AzureActiveDirectoryAccessToken...
Accept-Encoding: deflate
Content-Type: application/json; charset=utf-8
Host: help.kusto.windows.net
x-ms-client-request-id: MyApp.Query;e9f884e4-90f0-404a-8e8b-01d883023bf1
x-ms-user-id: EARTH\davidbg
x-ms-app: MyApp
```

Cuerpo de la solicitud

```json
{
  "db":"Samples",
  "csl":"print Test=\"Hello, World!\"",
  "properties":"{\"Options\":{\"queryconsistency\":\"strongconsistency\"},\"Parameters\":{},\"ClientRequestId\":\"MyApp.Query;e9f884e4-90f0-404a-8e8b-01d883023bf1\"}"
}
```

En este ejemplo se muestra cómo crear una solicitud que envía la consulta anterior, con [rizo](https://curl.haxx.se/).

1. Obtener un token para la autenticación.

    Reemplace `AAD_TENANT_NAME_OR_ID` , `AAD_APPLICATION_ID` y `AAD_APPLICATION_KEY` por los valores pertinentes, después de haber configurado la [autenticación de la aplicación AAD](../../management/access-control/how-to-provision-aad-app.md) .

    ```
    curl "https://login.microsoftonline.com/AAD_TENANT_NAME_OR_ID/oauth2/token" \
      -F "grant_type=client_credentials" \
      -F "resource=https://help.kusto.windows.net" \
      -F "client_id=AAD_APPLICATION_ID" \
      -F "client_secret=AAD_APPLICATION_KEY"
    ```

    Este fragmento de código le proporcionará el token de portador.

    ```
    {
      "token_type": "Bearer",
      "expires_in": "3599",
      "ext_expires_in":"3599", 
      "expires_on":"1578439805",
      "not_before":"1578435905",
      "resource":"https://help.kusto.windows.net",
      "access_token":"eyJ0...uXOQ"
    }
    ```

1. Use el token de portador en la solicitud al punto de conexión de la consulta.

    ```
    curl -d '{"db":"Samples","csl":"print Test=\"Hello, World!\"","properties":"{\"Options\":{\"queryconsistency\":\"strongconsistency\"}}"}"' \
    -H "Accept: application/json" \
    -H "Authorization: Bearer eyJ0...uXOQ" \
    -H "Content-Type: application/json; charset=utf-8" \
    -H "Host: help.kusto.windows.net" \
    -H "x-ms-client-request-id: MyApp.Query;e9f884e4-90f0-404a-8e8b-01d883023bf1" \
    -H "x-ms-user-id: EARTH\davidbg" \
    -H "x-ms-app: MyApp" \
    -X POST https://help.kusto.windows.net/v2/rest/query
    ```

1. Lea la respuesta según [esta especificación](response.md).
