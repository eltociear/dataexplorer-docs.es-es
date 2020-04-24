---
title: 'Solicitud HTTP de ingesta de streaming: Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la solicitud HTTP de ingesta de streaming en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: d14987806bbc62dbc79112700bd5b88aaa6676c3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81503016"
---
# <a name="streaming-ingestion-http-request"></a>Solicitud HTTP de ingesta de streaming

## <a name="request-verb-and-resource"></a>Solicitar verbo y recurso

|Acción    |Verbo HTTP|Recurso HTTP                                               |
|----------|---------|------------------------------------------------------------|
|Ingesta    |POST     |`/v1/rest/ingest/{database}/{table}?{additional parameters}`|

## <a name="request-parameters"></a>Parámetros de solicitud

| Parámetro    |  Descripción                                                                                                |
|--------------|-------------------------------------------------------------------------------------------------------------|
| `{database}` | **Requerido** Nombre de la base de datos de destino para la solicitud de ingesta                                          |
| `{table}`    | **Requerido** Nombre de la tabla de destino para la solicitud de ingesta                                             |

## <a name="additional-parameters"></a>Parámetros adicionales
Los parámetros adicionales tienen el formato `{name}` = `{value}` de consulta URL: pares separtido por & carácter


| Parámetro    |  Descripción                                                                                                |
|--------------|-------------------------------------------------------------------------------------------------------------|
|`streamFormat`| **Requerido** Especifica el formato de los datos en el cuerpo de la solicitud. El valor debe ser `Csv`uno`Tsv``Scsv`de`SOHsv``Psv`los`Json``SingleJson`siguientes: , , , , , , ,`MultiJson`.`Avro` Para obtener más información, consulte [Formatos de datos compatibles](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats).|
|`mappingName` | **Necesario** `streamFormat` si es `Json` `SingleJson`uno `MultiJson` `Avro`de , , o , **Opcional** en caso contrario. El valor debe ser el nombre de la asignación de ingesta creada previamente definida en la tabla. Para obtener más información sobre las asignaciones de datos, consulte [Asignaciones de](../../management/mappings.md)datos . La forma de administrar las asignaciones creadas [previamente](../../management/create-ingestion-mapping-command.md) en la tabla se describe aquí |
              

Por ejemplo, para ingerir datos `Logs` con `Test` formato CSV en la tabla de la base de datos, utilice la siguiente línea de solicitud:

```
POST https://help.kusto.windows.net/v1/rest/ingest/Test/Logs?streamFormat=Csv HTTP/1.1
```

para ingerir datos con formato JSON con mapeo creado previamente`mylogmapping`

```
POST https://help.kusto.windows.net/v1/rest/ingest/Test/Logs?streamFormat=Json&mappingName=mylogmapping HTTP/1.1
```


(Consulte a continuación los encabezados y el cuerpo de la solicitud que se incluirán.)

## <a name="request-headers"></a>Encabezados de solicitud

La tabla siguiente contiene los encabezados comunes utilizados para realizar operaciones de consulta y administración.

|Encabezado estándar  |Descripción                                                                                                              |
|------------------|------------------------------------------------------------------------------------------------------------------------|
|`Accept`          |**Opcional**. Establezca esta opción en `application/json`.                                                                           |
|`Accept-Encoding` |**Opcional**. Las codificaciones `gzip` `deflate`admitidas son y .                                                             |
|`Authorization`   |**Obligatorio**. Consulte [Autenticación](./authentication.md).                                                                |
|`Connection`      |**Opcional**. Se recomienda `Keep-Alive` que se habilite.                                                           |
|`Content-Length`  |**Opcional**. Se recomienda especificar la longitud del cuerpo de la solicitud cuando se conozca.                                   |
|`Content-Encoding`|**Opcional**. Se puede `gzip` ajustar en cuyo caso se requiere que el cuerpo se comprima con gzip                                 |
|`Expect`          |**Opcional**. Se puede `100-Continue`establecer en .                                                                             |
|`Host`            |**Obligatorio**. Establezca este nombre de dominio completo al que se envió `help.kusto.windows.net`la solicitud (por ejemplo, ).|

La tabla siguiente contiene los encabezados personalizados comunes que se usan al realizar operaciones de consulta y administración. A menos que se indique lo contrario, estos encabezados se usan únicamente con fines de telemetría y no tienen ningún impacto en la funcionalidad.

Todos los encabezados son **opcionales.** Sin embargo, se `x-ms-client-request-id` **recomienda encarecidamente** que se especifique el encabezado personalizado. En algunos escenarios (por ejemplo, cancelar una consulta en ejecución) este encabezado es **obligatorio** ya que se utiliza para identificar la solicitud.


|Encabezado personalizado           |Descripción                                                                                               |
|------------------------|----------------------------------------------------------------------------------------------------------|
|`x-ms-app`              |El nombre (amistoso) de la aplicación que realiza la solicitud.                                                |
|`x-ms-user`             |El nombre (descriptivo) del usuario que realiza la solicitud.                                                       |
|`x-ms-user-id`          |Igual a `x-ms-user`.                                                                                      |
|`x-ms-client-request-id`|Identificador único de la solicitud.                                                                      |
|`x-ms-client-version`   |Identificador de versión (descriptivo) para el cliente que realiza la solicitud.                                      |

## <a name="body"></a>Body

El cuerpo son los datos reales que se deben ingerir. Los formatos textuales shoud utilizan codificación UTF-8.

## <a name="examples"></a>Ejemplos

En el ejemplo siguiente se muestra la solicitud HTTP POST para un contenido JSON de ingesta:

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

En el ejemplo siguiente se muestra la solicitud HTTP POST para ingeniar los mismos datos comprimidos

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