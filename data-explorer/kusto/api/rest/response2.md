---
title: 'Respuesta HTTP de consulta V2: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la respuesta HTTP de Query V2 en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/11/2020
ms.openlocfilehash: 86a56d77005b2c6b5c9d38bbec85eebfbcb481dc
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617908"
---
# <a name="query-v2-http-response"></a>Respuesta HTTP de consulta V2

Si el código de estado es 200, el cuerpo de la respuesta es una matriz JSON.
Cada objeto JSON de la matriz se denomina _marco_.

Hay varios tipos de marcos:

* [DataSetHeader](#datasetheader)
* [TableHeader](#tableheader)
* [TableFragment](#tablefragment)
* [TableProgress](#tableprogress)
* [TableCompletion](#tablecompletion)
* [DataTable](#datatable)
* [DataSetCompletion](#datasetcompletion)

## <a name="datasetheader"></a>DataSetHeader 

El `DataSetHeader` marco siempre es el primero en el conjunto de datos y aparece exactamente una vez.

```json
{
    "Version": string,
    "IsProgressive": Boolean
}
```

Donde:

* `Version`es la versión del protocolo. La versión actual es `v2.0`.
* `IsProgressive`es una marca booleana que indica si este conjunto de datos contiene tramas progresivas. 
   Un fotograma progresivo es uno de los siguientes:
   
     | Fotograma             | Descripción                                    |
     |-------------------| -----------------------------------------------|
     | `TableHeader`     | Contiene información general sobre la tabla   |
     | `TableFragment`   | Contiene una partición de datos rectangular de la tabla |
     | `TableProgress`   | Contiene el progreso en porcentaje (0-100)       |
     | `TableCompletion` | Indica que este marco es el último      |
        
    Los marcos anteriores describen una tabla.
    Si la `IsProgressive` marca no está establecida en true, todas las tablas del conjunto se serializarán con un solo fotograma:
* `DataTable`: Contiene toda la información que el cliente necesita sobre una sola tabla del conjunto de datos.

## <a name="tableheader"></a>TableHeader

Las consultas que se realizan con `results_progressive_enabled` la opción establecida en true pueden incluir este marco. A continuación de esta tabla, los clientes pueden esperar una secuencia `TableFragment` de `TableProgress` intercalación de fotogramas y. El marco final de la tabla es `TableCompletion`.

```json
{
    "TableId": Number,
    "TableKind": string,
    "TableName": string,
    "Columns": Array,
}
```

Donde:

* `TableId`es el identificador único de la tabla.
* `TableKind`es uno de los siguientes:

    * PrimaryResult
    * QueryCompletionInformation
    * QueryTraceLog
    * QueryPerfLog
    * TableOfContents
    * QueryProperties
    * QueryPlan
    * Desconocido
      
* `TableName`es el nombre de la tabla.
* `Columns`es una matriz que describe el esquema de la tabla.

```json
{
    "ColumnName": string,
    "ColumnType": string,
}
```

Los tipos de columna admitidos se describen [aquí](../../query/scalar-data-types/index.md).

## <a name="tablefragment"></a>TableFragment

El `TableFragment` marco contiene un fragmento de datos rectangular de la tabla. Además de los datos reales, este marco también contiene una `TableFragmentType` propiedad que indica al cliente qué hacer con el fragmento. El fragmento se anexa a los fragmentos existentes o los reemplaza.

```json
{
    "TableId": Number,
    "FieldCount": Number,
    "TableFragmentType": string,
    "Rows": Array
}
```

Donde:

* `TableId`es el identificador único de la tabla.
* `FieldCount`es el número de columnas de la tabla.
* `TableFragmentType`describe lo que el cliente debe hacer con este fragmento. 
    `TableFragmentType`es uno de los siguientes:
    
    * Datos anexados
    * Reemplazar
      
* `Rows`es una matriz bidimensional que contiene los datos del fragmento.

## <a name="tableprogress"></a>TableProgress

El `TableProgress` marco puede intercalar `TableFragment` con el fotograma descrito anteriormente.
Su única finalidad es notificar al cliente el progreso de la consulta.

```json
{
    "TableId": Number,
    "TableProgress": Number,
}
```

Donde:

* `TableId`es el identificador único de la tabla.
* `TableProgress`es el progreso en porcentaje (0--100).

## <a name="tablecompletion"></a>TableCompletion

El `TableCompletion` marco marca el final de la transmisión de la tabla. No se enviarán más tramas relacionadas con esa tabla.

```json
{
    "TableId": Number,
    "RowCount": Number,
}
```    

Donde:

* `TableId`es el identificador único de la tabla.
* `RowCount`es el número total de filas de la tabla.

## <a name="datatable"></a>DataTable

Las consultas que se emiten `EnableProgressiveQuery` con la marca establecida en false no incluyen ninguno de los`TableHeader`Marcos `TableFragment`( `TableProgress`,, `TableCompletion`y). En su lugar, cada tabla del conjunto de datos se transmitirá usando `DataTable` el marco que contiene toda la información que el cliente necesita, para leer la tabla.

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

* `TableId`es el identificador único de la tabla.
* `TableKind`es uno de los siguientes:

    * PrimaryResult
    * QueryCompletionInformation
    * QueryTraceLog
    * QueryPerfLog
    * QueryProperties
    * QueryPlan
    * Desconocido
      
* `TableName`es el nombre de la tabla.
* `Columns`es una matriz que describe el esquema de la tabla e incluye:

```json
{
    "ColumnName": string,
    "ColumnType": string,
}
```

* `Rows`es una matriz bidimensional que contiene los datos de la tabla.

### <a name="the-meaning-of-tables-in-the-response"></a>El significado de las tablas en la respuesta

* `PrimaryResult`: Resultado tabular principal de la consulta. Para cada [instrucción de expresión tabular](../../query/tabularexpressionstatements.md), se generan una o varias tablas en orden, que representan los resultados generados por la instrucción. Puede haber varias tablas de este tipo por [lotes](../../query/batches.md) y [operadores de bifurcación](../../query/forkoperator.md).
* `QueryCompletionInformation`: Proporciona información adicional sobre la ejecución de la propia consulta, como si se completó correctamente o no, y cuáles fueron los recursos consumidos por la consulta (similar a la tabla QueryStatus en la respuesta V1). 
* `QueryProperties`: Proporciona valores adicionales, como instrucciones de visualización de cliente (emitidas, por ejemplo, para reflejar la información en el [operador de representación](../../query/renderoperator.md)) e información de cursor de [base de datos](../../management/databasecursor.md) .
* `QueryTraceLog`-La información de registro de seguimiento de rendimiento `perftrace` (devuelta cuando en [propiedades de solicitud de cliente](../netfx/request-properties.md) está establecida en true).

## <a name="datasetcompletion"></a>DataSetCompletion

El `DataSetCompletion` marco es el final del conjunto de datos.

```json
{
    "HasErrors": Boolean,
    "Cancelled": Boolean,
    "OneApiErrors": Array,
}
```

Donde:

* `HasErrors`es true si se produjeron errores al generar el conjunto de datos.
* `Cancelled`es true si la solicitud que condujo a la generación del conjunto de datos se canceló antes de la finalización. 
* `OneApiErrors`solo se devuelve si `HasErrors` es true. Para obtener una descripción del `OneApiErrors` formato, vea la sección 7.10.2 [aquí](https://github.com/Microsoft/api-guidelines/blob/vNext/Guidelines.md).
