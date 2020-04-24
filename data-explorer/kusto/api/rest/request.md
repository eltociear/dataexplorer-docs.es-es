---
title: Solicitud HTTP de consulta/administración- Azure Data Explorer Microsoft Docs
description: En este artículo se describe la solicitud HTTP de consulta y administración en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/27/2020
ms.openlocfilehash: 71fac7122ca51755beaa09ce3867143806e4f2b3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81502727"
---
# <a name="querymanagement-http-request"></a>Solicitud HTTP de consulta/administración

## <a name="request-verb-and-resource"></a>Solicitar verbo y recurso

|Acción    |Verbo HTTP|Recurso HTTP   |
|----------|---------|----------------|
|Consultar     |GET      |`/v1/rest/query`|
|Consultar     |POST     |`/v1/rest/query`|
|Consulta v2  |GET      |`/v2/rest/query`|
|Consulta v2  |POST     |`/v2/rest/query`|
|Administración|POST     |`/v1/rest/mgmt` |

Por ejemplo, para enviar un comando de control ("administración") a un punto de conexión de servicio, utilice la siguiente línea de solicitud:

```
POST https://help.kusto.windows.net/v1/rest/mgmt HTTP/1.1
```

(Consulte a continuación los encabezados y el cuerpo de la solicitud que se incluirán.)

## <a name="request-headers"></a>Encabezados de solicitud

La tabla siguiente contiene los encabezados comunes utilizados para realizar operaciones de consulta y administración.

|Encabezado estándar  |Descripción                                                                                                                    |
|-----------------|-------------------------------------------------------------------------------------------------------------------------------|
|`Accept`         |**Obligatorio**. Establezca esta opción en `application/json`.                                                                                  |
|`Accept-Encoding`|**Opcional**. Las codificaciones `gzip` `deflate`admitidas son y .                                                                    |
|`Authorization`  |**Obligatorio**. Consulte [Autenticación](./authentication.md).                                                                       |
|`Connection`     |**Opcional**. Se recomienda `Keep-Alive` que se habilite.                                                                  |
|`Content-Length` |**Opcional**. Se recomienda especificar la longitud del cuerpo de la solicitud cuando se conozca.                                          |
|`Content-Type`   |**Obligatorio**. Establezca esta `application/json` `charset=utf-8`configuración en con .                                                             |
|`Expect`         |**Opcional**. Se puede `100-Continue`establecer en .                                                                                    |
|`Host`           |**Obligatorio**. Establezca este nombre de dominio completo al que se envió la solicitud (por ejemplo, `help.kusto.windows.net`).|

La tabla siguiente contiene los encabezados personalizados comunes que se usan al realizar operaciones de consulta y administración. A menos que se indique lo contrario, estos encabezados se usan únicamente con fines de telemetría y no tienen ningún impacto en la funcionalidad.

Todos los encabezados son **opcionales.** Se **recomienda encarecidamente** que `x-ms-client-request-id` especifique el encabezado personalizado. En algunos escenarios (por ejemplo, cancelar una consulta en ejecución) este encabezado es **obligatorio** porque se usa para identificar la solicitud.


|Encabezado personalizado           |Descripción                                                                                               |
|------------------------|----------------------------------------------------------------------------------------------------------|
|`x-ms-app`              |El nombre (amistoso) de la aplicación que realiza la solicitud.                                                |
|`x-ms-user`             |El nombre (descriptivo) del usuario que realiza la solicitud.                                                       |
|`x-ms-user-id`          |Igual a `x-ms-user`.                                                                                      |
|`x-ms-client-request-id`|Identificador único de la solicitud.                                                                      |
|`x-ms-client-version`   |Identificador de versión (descriptivo) para el cliente que realiza la solicitud.                                      |
|`x-ms-readonly`         |Si se especifica, obliga a que la solicitud se ejecute en modo de solo lectura (impidiendo que realice cambios duraderos).|

## <a name="request-parameters"></a>Parámetros de solicitud

Los siguientes parámetros se pueden pasar en la solicitud. Se codifican en la solicitud como parámetros de consulta o como parte del cuerpo, dependiendo de si se utiliza GET o POST.

* `csl`: Este es un parámetro **obligatorio.** Contiene el texto del comando de consulta o control que se va a ejecutar.

* `db`: este es un parámetro **opcional** para algunos comandos de control y parámetro **obligatorio** para otros comandos de control y todas las consultas. Proporciona el nombre de la "base de datos en el ámbito": la base de datos que es el destino del comando de consulta o control.

* `properties`: Este es un parámetro **opcional.** Proporciona varias propiedades de solicitud de cliente que modifican cómo se procesa la solicitud y sus resultados. Para obtener una descripción completa, consulte Propiedades de solicitud de [cliente](../netfx/request-properties.md).

## <a name="get-query-parameters"></a>Parámetros de consulta GET

Cuando se utiliza GET, los parámetros de consulta de la solicitud especifican los parámetros de solicitud indicados anteriormente.

## <a name="body"></a>Body

Cuando se utiliza POST, el cuerpo de la solicitud es un único documento JSON codificado en UTF-8 que proporciona los valores de los parámetros de solicitud indicados anteriormente.

## <a name="examples"></a>Ejemplos

En el ejemplo siguiente se muestra la solicitud HTTP POST para una consulta:

```txt
POST https://help.kusto.windows.net/v2/rest/query HTTP/1.1
```

Encabezados de solicitud:

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

Cuerpo de solicitud (nuevas líneas introducidas para mayor claridad; no son necesarias):

```json
{
  "db":"Samples",
  "csl":"print Test=\"Hello, World!\"",
  "properties":"{\"Options\":{\"queryconsistency\":\"strongconsistency\"},\"Parameters\":{},\"ClientRequestId\":\"MyApp.Query;e9f884e4-90f0-404a-8e8b-01d883023bf1\"}"
}
```

En el ejemplo siguiente se muestra cómo crear una solicitud que envía la consulta anterior mediante [curl:](https://curl.haxx.se/)

1. Obtenga un token para la autenticación.

* reemplazar `AAD_TENANT_NAME_OR_ID` `AAD_APPLICATION_ID` , `AAD_APPLICATION_KEY` y con los valores relevantes, después de haber configurado la autenticación de [aplicaciones de AAD](../../management/access-control/how-to-provision-aad-app.md)

```
curl "https://login.microsoftonline.com/AAD_TENANT_NAME_OR_ID/oauth2/token" \
  -F "grant_type=client_credentials" \
  -F "resource=https://help.kusto.windows.net" \
  -F "client_id=AAD_APPLICATION_ID" \
  -F "client_secret=AAD_APPLICATION_KEY"
```

Esto le proporcionará el token de portador:

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

2. Utilice el token de portador en su solicitud al punto de conexión de consulta:

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

3. Lea la respuesta de acuerdo con [esta especificación.](response.md)