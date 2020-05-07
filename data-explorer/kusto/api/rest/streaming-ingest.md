---
title: 'Solicitud HTTP de ingesta de streaming: Azure Explorador de datos'
description: En este artículo se describe la solicitud HTTP de ingesta de streaming en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 672f924865cab14dff6c7d5319c3c34cca1a67ee
ms.sourcegitcommit: f6cf88be736aa1e23ca046304a02dee204546b6e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2020
ms.locfileid: "82862025"
---
# <a name="streaming-ingestion-http-request"></a>Solicitud HTTP de ingesta de streaming

## <a name="request-verb-and-resource"></a>Recurso y verbo de solicitud

|Acción    |Verbo HTTP|Recurso HTTP                                               |
|----------|---------|------------------------------------------------------------|
|Ingesta    |POST     |`/v1/rest/ingest/{database}/{table}?{additional parameters}`|

## <a name="request-parameters"></a>Parámetros de solicitud

| Parámetro    | Descripción                                                                 | Obligatorio/opcional |
|--------------|-----------------------------------------------------------------------------|-------------------|
| `{database}` |   Nombre de la base de datos de destino para la solicitud de ingesta                     |  Obligatorio         |
| `{table}`    |   Nombre de la tabla de destino para la solicitud de ingesta                        |  Obligatorio         |

## <a name="additional-parameters"></a>Parámetros adicionales

Los parámetros adicionales se formatean como `{name}={value}` pares de consulta de dirección URL, separados por el carácter &.

| Parámetro    | Descripción                                                                          | Obligatorio/opcional   |
|--------------|--------------------------------------------------------------------------------------|---------------------|
|`streamFormat`| Especifica el formato de los datos en el cuerpo de la solicitud. El valor debe ser uno de los `CSV`siguientes`TSV`:`SCsv`,`SOHsv`,`PSV`,`JSON`,`SingleJSON`,`MultiJSON`,`Avro`,,. Para obtener más información, vea [formatos de datos admitidos](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats).| Obligatorio |
|`mappingName` | Nombre de la asignación de ingesta creada previamente definida en la tabla. Para obtener más información, vea [asignaciones de datos](../../management/mappings.md). La forma de administrar las asignaciones creadas previamente en la tabla se describe [aquí](../../management/create-ingestion-mapping-command.md).| Opcional, pero necesario si `streamFormat` es `JSON`,`SingleJSON`,`MultiJSON`o`Avro`|  |
              
Por ejemplo, para introducir datos con formato CSV en una `Logs` tabla de `Test`la base de datos, use:

```
POST https://help.kusto.windows.net/v1/rest/ingest/Test/Logs?streamFormat=Csv HTTP/1.1
```

Para ingerir datos con formato JSON con una asignación `mylogmapping`creada previamente, use:

```
POST https://help.kusto.windows.net/v1/rest/ingest/Test/Logs?streamFormat=Json&mappingName=mylogmapping HTTP/1.1
```

## <a name="request-headers"></a>Encabezados de solicitud

La tabla siguiente contiene los encabezados comunes para las operaciones de consulta y administración.

|Encabezado estándar   | Descripción                                                                               | Obligatorio/opcional | 
|------------------|-------------------------------------------------------------------------------------------|-------------------|
|`Accept`          | Establezca este valor en `application/json`.                                                     | Opcional          |
|`Accept-Encoding` | Las codificaciones admitidas `gzip` son `deflate`y.                                             | Opcional          | 
|`Authorization`   | Vea [autenticación](./authentication.md).                                                | Obligatorio          |
|`Connection`      | Habilite `Keep-Alive`.                                                                      | Opcional          |
|`Content-Length`  | Especifique la longitud del cuerpo de la solicitud, cuando se conozca.                                              | Opcional          |
|`Content-Encoding`| Establecido en `gzip` , pero el cuerpo debe estar comprimido con gzip                                        | Opcional          |
|`Expect`          | Establézcalo en `100-Continue`.                                                                    | Opcional          |
|`Host`            | Establezca en el nombre de dominio al que envió la solicitud (por ejemplo, `help.kusto.windows.net`). | Obligatorio          |

La tabla siguiente contiene los encabezados personalizados comunes para las operaciones de consulta y administración. A menos que se indique lo contrario, los encabezados solo tienen fines de telemetría y no tienen ningún impacto en la funcionalidad.

|Encabezado personalizado           |Descripción                                                                           | Obligatorio/opcional |
|------------------------|----------------------------------------------------------------------------------------------------------|
|`x-ms-app`              |Nombre (descriptivo) de la aplicación que realiza la solicitud.                            | Opcional          |
|`x-ms-user`             |Nombre (descriptivo) del usuario que realiza la solicitud.                                   | Opcional          |
|`x-ms-user-id`          |Igual a `x-ms-user`.                                                                  | Opcional          |
|`x-ms-client-request-id`|Identificador único de la solicitud.                                                  | Opcional          |
|`x-ms-client-version`   |Identificador de versión (descriptivo) para el cliente que realiza la solicitud. Se requiere en escenarios, donde se usa para identificar la solicitud, como cancelar una consulta en ejecución.                                                        | Opcional/?Requerido  |

## <a name="body"></a>Body

El cuerpo son los datos reales que se van a ingerir. Los formatos de texto deben usar la codificación UTF-8.

## <a name="examples"></a>Ejemplos

En el ejemplo siguiente se muestra la solicitud HTTP POST para la ingesta de contenido JSON:

```txt
POST https://help.kusto.windows.net/v1/rest/ingest/Test/Logs?streamFormat=Json&mappingName=mylogmapping HTTP/1.1
```

Encabezados de solicitud:

```txt
Authorization: Bearer ...AzureActiveDirectoryAccessToken...
Accept-Encoding: deflate
Accept-Encoding: gzip
Connection: Keep-Alive
Content-Length: 161
Host: help.kusto.windows.net
x-ms-client-request-id: MyApp.Ingest;5c0656b9-37c9-4e3a-a671-5f83e6843fce
x-ms-user-id: alex@contoso.com
x-ms-app: MyApp
```

Cuerpo de la solicitud:

```txt
{"Timestamp":"2018-11-14 11:34","Level":"Info","EventText":"Nothing Happened"}
{"Timestamp":"2018-11-14 11:35","Level":"Error","EventText":"Something Happened"}
```

En el ejemplo siguiente se muestra la solicitud HTTP POST para ingerir los mismos datos comprimidos.

```txt
POST https://help.kusto.windows.net/v1/rest/ingest/Test/Logs?streamFormat=Json&mappingName=mylogmapping HTTP/1.1
```

Encabezados de solicitud:

```txt
Authorization: Bearer ...AzureActiveDirectoryAccessToken...
Accept-Encoding: deflate
Accept-Encoding: gzip
Connection: Keep-Alive
Content-Length: 116
Content-Encoding: gzip
Host: help.kusto.windows.net
x-ms-client-request-id: MyApp.Ingest;5c0656b9-37c9-4e3a-a671-5f83e6843fce
x-ms-user-id: alex@contoso.com
x-ms-app: MyApp
```

Cuerpo de la solicitud:

```
... binary data ...
```
