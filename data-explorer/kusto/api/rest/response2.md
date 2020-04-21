---
title: 'Consulta de respuesta HTTP V2: Explorador de azure data Microsoft Docs'
description: En este artículo se describe la respuesta HTTP de consulta V2 en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: cca9b8381c7c59993c1e9071c46f34c1754d2429
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81524317"
---
# <a name="query-v2-http-response"></a>Consulta de respuesta HTTP V2

Si el código de estado es 200, el cuerpo de la respuesta es una matriz JSON.
Cada objeto JSON de la matriz se denomina _marco._

Hay 7 tipos de marcos:

1. DataSetHeader
2. TableHeader
3. TableFragment
4. TableProgress
5. TableCompletion
6. DataTable
7. DataSetCompletion

## <a name="datasetheader"></a>DataSetHeader 

Este es siempre el primer fotograma del conjunto de datos y aparece exactamente una vez.

```json
{
    "Version": string,
    "IsProgressive": Boolean
}
```

Donde:

1. `Version`contiene la versión del protocolo. La versión actual es `v2.0`.
2. `IsProgressive`es un indicador booleano que indica si este conjunto de datos contiene marcos progresivos. Un marco progresivo es uno de los siguientes:
    1. `TableHeader`: Contiene información general sobre la tabla
    2. `TableFragment`: Contiene una partición de datos rectangluar de la tabla
    3. `TableProgress`: Contiene el progreso en porcentaje (0-100)
    4. `TableCompletion`: marca que este es el último fotograma de la tabla.
        
    Los cuatro fotogramas anteriores se utilizan juntos para describir una tabla.
    Si no se establece esta marca, todas las tablas del conjunto se serializarán con un solo fotograma:
      1. `DataTable`: contiene toda la información que el cliente necesita sobre una sola tabla en el conjunto de datos.


## <a name="tableheader"></a>TableHeader

Las consultas que `results_progressive_enabled` se emiten con la opción establecida en true pueden incluir este marco. Siguiendo esta tabla, los clientes `TableFragment` `TableProgress` deben esperar una `TableCompletion` secuencia de entrelazado de y fotogramas, seguido de una trama. Después de lo cual, no se enviarán más fotogramas relacionados con esa tabla.

```json
{
    "TableId": Number,
    "TableKind": string,
    "TableName": string,
    "Columns": Array,
}
```

Donde:

1. `TableId`es el identificador único de la mesa.
2. `TableKind`es el tipo de la mesa, que puede ser uno de los siguientes:

      * PrimaryResult
      * QueryCompletionInformation
      * QueryTraceLog
      * QueryPerfLog
      * TableOfContents
      * QueryProperties
      * QueryPlan
      * Desconocido
3. `TableName`es el nombre de la mesa.
4. `Columns`es una matriz que describe el esquema de la tabla:

```json
{
    "ColumnName": string,
    "ColumnType": string,
}
```
Los tipos de columna admitidos se describen [aquí.](../../query/scalar-data-types/index.md)

## <a name="tablefragment"></a>TableFragment

Este marco contiene un fragmento de datos rectangular de la tabla. Además de los datos reales, este marco contiene una `TableFragmentType` propiedad, que indica al cliente qué hacer con el fragmento (se puede anexar a fragmentos existentes o reemplazarlos todos juntos).

```json
{
    "TableId": Number,
    "FieldCount": Number,
    "TableFragmentType": string,
    "Rows": Array
}
```

Donde:

1. `TableId`es el identificador único de la mesa.
2. `FieldCount`es el número de columnas en la tabla
3. `TableFragmentType`describe lo que el cliente debe hacer con este fragmento. Puede ser uno de los siguientes:
      * DataAppend
      * DataReplace
4. `Rows`es una matriz bidimensional que contiene los datos del fragmento.

## <a name="tableprogress"></a>TableProgress

Este marco se puede `TableFragment` intercalar con el marco descrito anteriormente.
Su único propósito es notificar al cliente sobre el progreso de la consulta.

```json
{
    "TableId": Number,
    "TableProgress": Number,
}
```

Donde:

1. `TableId`es el identificador único de la mesa.
2. `TableProgress`es el progreso en porcentaje (0-100).

## <a name="tablecompletion"></a>TableCompletion

Las `TableCompletion` tramas marcan el final de la transmisión de la tabla. No se enviarán más fotogramas relacionados con esa tabla.

```json
{
    "TableId": Number,
    "RowCount": Number,
}
```    

Donde:

1. `TableId`es el identificador único de la mesa.
2. `RowCount`es el número final de filas de la tabla.

## <a name="datatable"></a>DataTable

Las consultas que `EnableProgressiveQuery` se emitan con el indicador establecido en`TableHeader` `TableFragment`false `TableProgress` `TableCompletion`no incluirán ninguno de los 4 fotogramas anteriores ( , , y ). En su lugar, cada tabla en el conjunto de `DataTable` datos se transmitirá usando una sola trama, la trama, que contiene toda la información que el cliente necesita para leer la tabla.

```json
{
    "TableId": Number,

    "TableKind": string,

    "TableName": string,

    "Columns": Array,

    "Rows": Array,
}
```    

Donde:

1. `TableId`es el identificador único de la mesa.
2. `TableKind`es el tipo de la mesa, que puede ser uno de los siguientes:

      * PrimaryResult
      * QueryCompletionInformation
      * QueryTraceLog
      * QueryPerfLog
      * QueryProperties
      * QueryPlan
      * Desconocido
3. `TableName`es el nombre de la mesa.
4. `Columns`es una matriz que describe el esquema de la tabla:

```json
{
    "ColumnName": string,
    "ColumnType": string,
}
```
4. `Rows`es una matriz bidimensional que contiene los datos de la tabla.

### <a name="the-meaning-of-tables-in-the-response"></a>El significado de las tablas en la respuesta

* `PrimaryResult`- El principal resultado tabular de la consulta. Para cada [instrucción](../../query/tabularexpressionstatements.md)de expresión tabular , una o más tablas se emiten en orden, que representan los resultados generados por la instrucción (puede haber varias de estas tablas debido a [lotes](../../query/batches.md) y operadores de [bifurcación).](../../query/forkoperator.md)
* `QueryCompletionInformation`: proporciona información adicional sobre la ejecución de la propia consulta, como si se completó correctamente o no y cuáles fueron los recursos consumidos por la consulta (similar a la tabla QueryStatus de la respuesta v1). 
* `QueryProperties`- proporciona valores adicionales como instrucciones de visualización del cliente (emitidas, por ejemplo, para reflejar la información en el operador de [representación)](../../management/databasecursor.md) y la información del cursor de la base de datos). [render operator](../../query/renderoperator.md)
* `QueryTraceLog`- información de registro de seguimiento de rendimiento (devuelta al establecer perftrace en las propiedades de solicitud de [cliente).](../netfx/request-properties.md)

## <a name="datasetcompletion"></a>DataSetCompletion

Este es el fotograma final del conjunto de datos.
```json
{
    "HasErrors": Boolean,
    "Cancelled": Boolean,
    "OneApiErrors": Array,
}
```

Donde:

1. `HasErrors`es cierto si hubo algún error al generar el conjunto de datos.
2. `Cancelled`es cierto si la solicitud que conduce a la generación del conjunto de datos se canceló a mitad de camino. 
3. `OneApiErrors`sólo se transmite `HasErrors` si es cierto. Para obtener una `OneApiErrors` descripción del formato, consulte la sección 7.10.2 [aquí](https://github.com/Microsoft/api-guidelines/blob/vNext/Guidelines.md).