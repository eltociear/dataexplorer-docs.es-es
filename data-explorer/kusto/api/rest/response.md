---
title: 'Respuesta HTTP de consulta/administración: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la respuesta HTTP de consulta/administración en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/05/2018
ms.openlocfilehash: 5a9166066ee664f15b07dff2b0a535044ebb929c
ms.sourcegitcommit: 061eac135a123174c85fe1afca4d4208c044c678
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/05/2020
ms.locfileid: "82799652"
---
# <a name="querymanagement-http-response"></a>Respuesta HTTP de consulta/administración

## <a name="response-status"></a>Estado de respuesta

La línea de estado de respuesta HTTP sigue los códigos de respuesta HTTP estándar.
Por ejemplo, el código 200 indica que la operación se ha realizado correctamente. 

Los siguientes códigos de estado se están usando actualmente, aunque se puede devolver cualquier código HTTP válido.

|Código|Subcódigo        |Descripción                                    |
|----|---------------|-----------------------------------------------|
|100 |Continuar       |El cliente puede continuar enviando la solicitud.       |
|200 |Aceptar             |La solicitud comenzó el procesamiento correctamente.       |
|400 |BadRequest     |La solicitud tiene un formato incorrecto y no se pudo realizar (de forma permanente).|
|401 |No autorizado   |El cliente debe autenticarse primero.            |
|403 |Prohibido      |Se denegó la solicitud de cliente.                      |
|404 |NotFound       |La solicitud hace referencia a una entidad no existente.      |
|413 |PayloadTooLarge|La carga de solicitud superó los límites.               |
|429 |TooManyRequests|Se denegó la solicitud debido a la limitación. |
|504 |Tiempo de espera        |La solicitud ha agotado el tiempo de espera.                         |
|520 |ServiceError   |El servicio encontró un error al procesar la solicitud.|

> [!NOTE]
> El código de estado 200 muestra que el procesamiento de la solicitud se ha iniciado correctamente y no que se ha completado correctamente.
> Los errores encontrados durante el procesamiento de la solicitud después de que se devuelva el código de estado 200 se denominan "errores de consulta parciales" y, cuando se encuentran, se insertan indicadores especiales en la secuencia de respuesta para avisar al cliente de que se produjeron.

## <a name="response-headers"></a>Encabezados de respuesta

Se devolverán los siguientes encabezados personalizados.

|Encabezado personalizado           |Descripción                                                                                               |
|------------------------|----------------------------------------------------------------------------------------------------------|
|`x-ms-client-request-id`|Identificador único de la solicitud enviado en el encabezado de solicitud con el mismo nombre o con un identificador único.     |
|`x-ms-activity-id`      |Identificador de correlación globalmente único para la solicitud. Lo crea el servicio.                    |

## <a name="response-body"></a>Cuerpo de la respuesta

Si el código de estado es 200, el cuerpo de la respuesta es un documento JSON que codifica los resultados del comando de consulta o control como una secuencia de tablas rectangulares.
Consulte a continuación para más información.

> [!NOTE]
> El SDK refleja la secuencia de tablas. Por ejemplo, al usar la biblioteca .NET Framework Kusto. Data, la secuencia de tablas se convierte entonces en el resultado `System.Data.IDataReader` del objeto devuelto por el SDK.

Si el código de estado indica un error 4xx o 5xx, que no sea 401, el cuerpo de la respuesta es un documento JSON que codifica los detalles del error.
Para más información, consulte las directrices de la [API de REST de Microsoft](https://github.com/microsoft/api-guidelines).

> [!NOTE]
> Si el `Accept` encabezado no se incluye con la solicitud, el cuerpo de la respuesta de un error no es necesariamente un documento JSON.

## <a name="json-encoding-of-a-sequence-of-tables"></a>Codificación JSON de una secuencia de tablas

La codificación JSON de una secuencia de tablas es un contenedor de propiedades JSON único con los siguientes pares de nombre y valor.

|Nombre  |Value                              |
|------|-----------------------------------|
|Tablas|Matriz de la bolsa de propiedades de tabla.|

La bolsa de propiedades de tabla tiene los siguientes pares de nombre/valor.

|Nombre     |Value                               |
|---------|------------------------------------|
|TableName|Cadena que identifica la tabla. |
|Columnas  |Matriz de la bolsa de propiedades de columna.|
|Filas     |Matriz de la matriz de filas.          |

La bolsa de propiedades de columna tiene los siguientes pares de nombre y valor.

|Nombre      |Value                                                          |
|----------|---------------------------------------------------------------|
|ColumnName|Cadena que identifica la columna.                           |
|DataType  |Una cadena que proporciona el tipo .NET aproximado de la columna.|
|ColumnType|Cadena que proporciona el [tipo de datos escalar](../../query/scalar-data-types/index.md) de la columna.|

La matriz de filas tiene el mismo orden que la matriz de columnas correspondiente.
La matriz de filas también tiene un elemento que coincide con el valor de la fila de la columna pertinente.
Los tipos de datos escalares que no se pueden representar en `datetime` JSON `timespan`, como y, se representan como cadenas JSON.

En el ejemplo siguiente se muestra un objeto posible, cuando contiene una sola tabla denominada `Table_0` que tiene una sola columna `Text` de tipo `string`y una sola fila.

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

Otro ejemplo: 

:::image type="content" source="../images/rest-json-representation.png" alt-text="Rest: representación JSON":::

## <a name="the-meaning-of-tables-in-the-response"></a>El significado de las tablas en la respuesta

En la mayoría de los casos, los comandos de control devuelven un resultado con una sola tabla que contiene la información generada por el comando de control. Por ejemplo, el `.show databases` comando devuelve una sola tabla con los detalles de todas las bases de datos accesibles en el clúster.

Las consultas normalmente devuelven varias tablas.
Para cada [instrucción de expresión tabular](../../query/tabularexpressionstatements.md), se generan una o varias tablas en orden, que representan los resultados generados por la instrucción.

> [!NOTE]
> Puede haber varias tablas de este tipo por [lotes](../../query/batches.md) y [operadores de bifurcación](../../query/forkoperator.md)).

A menudo se generan tres tablas:

* Una @ExtendedProperties tabla que proporciona valores adicionales, como instrucciones de visualización de cliente. Estos valores se generan, por ejemplo, para reflejar la información en el [operador de representación](../../query/renderoperator.md)y el cursor de base de [datos](../../management/databasecursor.md).
* Una tabla QueryStatus que proporciona información adicional sobre la ejecución de la propia consulta, como, si se ha completado correctamente o no, y cuáles son los recursos consumidos por la consulta.
* Una tabla TableOfContents, que se crea en último lugar y enumera las demás tablas de los resultados.
