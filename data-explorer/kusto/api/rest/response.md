---
title: 'Respuesta HTTP de consulta/administración: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la respuesta HTTP de consulta y administración en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/05/2018
ms.openlocfilehash: 13937bbc9d17ed570c40926497ccf9b5c942fe72
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81503084"
---
# <a name="querymanagement-http-response"></a>Respuesta HTTP de consulta/administración

## <a name="response-status"></a>Estado de respuesta

La línea de estado de respuesta HTTP sigue los códigos de respuesta estándar HTTP (por ejemplo, 200 indica que se ha realizado correctamente). Los siguientes códigos de estado están actualmente en uso (pero tenga en cuenta que se puede devolver cualquier código HTTP válido):

|Código|Subcódigo       |Descripción                                    |
|----|---------------|-----------------------------------------------|
|100 |Continuar       |El cliente puede continuar enviando la solicitud.       |
|200 |Aceptar             |La solicitud comenzó a procesarse correctamente.       |
|400 |BadRequest     |La solicitud está mal formada y falló (permanentemente).|
|401 |No autorizado   |El cliente necesita autenticarse primero.            |
|403 |Prohibido      |Se deniega la solicitud del cliente.                      |
|404 |NotFound       |La solicitud hace referencia a una entidad no existente.      |
|413 |PayloadTooLarge|La carga útil de la solicitud excedió los límites.               |
|429 |TooManyRequests|La solicitud se ha denegado debido a la limitación.     |
|504 |Tiempo de espera        |La solicitud ha transcurrido el tiempo de espera.                         |
|520 |ServiceError   |El servicio encontró un error al procesar la solicitud.|

> [!NOTE]
> Es importante tener en cuenta que el código de estado 200 representa que el procesamiento de solicitudes se ha iniciado correctamente y no que se ha completado correctamente. Los errores encontrados durante el procesamiento de solicitudes, pero después de devolver 200 se denominan "errores de consulta parciales", y cuando se encuentran se insertan indicadores especiales en la secuencia de respuesta para alertar al cliente de que se produjeron.

## <a name="response-headers"></a>Encabezados de respuesta

Se devolverán los siguientes encabezados personalizados.

|Encabezado personalizado           |Descripción                                                                                               |
|------------------------|----------------------------------------------------------------------------------------------------------|
|`x-ms-client-request-id`|Identificador de solicitud único enviado en el encabezado de solicitud del mismo nombre o algún identificador único.     |
|`x-ms-activity-id`      |Identificador de correlación único globalmente para la solicitud (creado por el servicio).                        |

## <a name="response-body"></a>Response body

Si el código de estado es 200, el cuerpo de la respuesta es un documento JSON que codifica los resultados del comando de consulta o control como una secuencia de tablas rectangulares.
Consulte a continuación para más información.

> [!NOTE]
> Esta secuencia de tablas se refleja en el SDK. Por ejemplo, cuando se usa la biblioteca Kusto.Data de .NET Framework, la secuencia de tablas se convierte en los resultados en el `System.Data.IDataReader` objeto devuelto por el SDK.

Si el código de estado indica un error 4xx o 5xx (que no sea 401), el cuerpo de la respuesta es un documento JSON que codifica los detalles del error, de acuerdo con las directrices de la API REST de [Microsoft.](https://github.com/microsoft/api-guidelines)

> [!NOTE]
> Si `Accept` el encabezado no se incluye con la solicitud, el cuerpo de respuesta de un error no es necesariamente un documento JSON.

## <a name="json-encoding-of-a-sequence-of-tables"></a>Codificación JSON de una secuencia de tablas

La codificación JSON de una secuencia de tablas es una única bolsa de propiedades JSON con los siguientes pares nombre/valor:

|NOMBRE  |Valor                              |
|------|-----------------------------------|
|Tablas|Matriz de la bolsa de propiedades Table.|

El contenedor de propiedades Table tiene los siguientes pares nombre/valor:

|NOMBRE     |Valor                               |
|---------|------------------------------------|
|TableName|Cadena que identifica la tabla. |
|Columnas  |Matriz de la bolsa de propiedades Column.|
|Filas     |Matriz de la matriz Row.          |

El contenedor de propiedades Column tiene los siguientes pares nombre/valor:

|NOMBRE      |Valor                                                          |
|----------|---------------------------------------------------------------|
|ColumnName|Cadena que identifica la columna.                           |
|DataType  |Cadena que proporciona el tipo de .NET aproximado de la columna.|
|ColumnType|Cadena que proporciona el tipo de [datos escalar](../../query/scalar-data-types/index.md) de la columna.|

La matriz Row tiene el mismo orden que la matriz Columns respectiva y tiene un elemento con el valor de la fila para la columna relevante.
Los tipos de datos escalares que no `datetime
and `se pueden representar en JSON (como timespan') se representan como cadenas JSON.

En el ejemplo siguiente se muestra un posible objeto `Table_0` de este tipo, cuando contiene una sola tabla llamada que contiene una sola columna `Text` de tipo `string` y una sola fila.

```json
{
    "Tables": [{
        "TableName": "Table_0",
        "Columns": [{
            "ColumnName": "Text",
            "DataType": "String",
            "ColumnType": "string"
        }],
        "Rows": [["Hello, World!"]]
}
```

Otro exmaple:

![Representación de respuesta JSON](../images/rest-json-representation.png "rest-json-representation")

## <a name="the-meaning-of-tables-in-the-response"></a>El significado de las tablas en la respuesta

En la mayoría de los casos, los comandos de control devuelven un resultado con una sola tabla, manteniendo la información generada por el comando de control. Por ejemplo, `.show databases` el comando devuelve una sola tabla con los detalles de todas las bases de datos accesobles del clúster.

Las consultas, por otro lado, generalmente devuelven varias tablas. Para cada [instrucción](../../query/tabularexpressionstatements.md)de expresión tabular , una o más tablas se emiten en orden, que representan los resultados generados por la instrucción (puede haber varias de estas tablas debido a [lotes](../../query/batches.md) y operadores de [bifurcación).](../../query/forkoperator.md)

Además, normalmente se producen tres tablas:

* Una @ExtendedProperties tabla, que proporciona valores adicionales como instrucciones de visualización de cliente (emitidas, por ejemplo, para reflejar la información en el operador de [representación)](../../management/databasecursor.md) y la información del cursor de base de datos). [render operator](../../query/renderoperator.md)
* Una tabla QueryStatus, que proporciona información adicional sobre la ejecución de la propia consulta, como si se completó correctamente o no, y cuáles fueron los recursos consumidos por la consulta.
* Una tabla TableOfContents, que se emite en último lugar y enumera las otras tablas de los resultados.

