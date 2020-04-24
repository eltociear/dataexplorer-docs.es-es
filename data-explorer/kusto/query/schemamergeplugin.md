---
title: schema_merge complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe schema_merge complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/16/2020
ms.openlocfilehash: 67326a0e7a92d064613ee3a3de2851addb502fc9
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509255"
---
# <a name="schema_merge-plugin"></a>schema_merge plugin

Combina definiciones de esquema tabular en esquema unificado. 

Se espera que las definiciones de esquema estén en formato según lo producido por el operador [getschema.](./getschemaoperator.md)

La operación de combinación de esquemas une columnas que aparecen en esquemas de entrada e intenta reducir los tipos de datos a un tipo de datos común (en caso de que los tipos de datos no se puedan reducir, se muestra un error en una columna problemática).

```kusto
let Schema1=Table1 | getschema;
let Schema2=Table2 | getschema;
union Schema1, Schema2 | evaluate schema_merge()
```

**Sintaxis**

`T``|` `evaluate` *PreserveOrder* PreserveOrder `schema_merge(``)`

**Argumentos**

* *PreserveOrder*: (Opcional) `true`Cuando se establece en , se dirige al complemento para validar ese orden de columna tal como se define en el primer esquema tabular se mantiene. En otras palabras, si la misma columna aparece en varios esquemas, el ordinal de columna debe ser como en el primer esquema que apareció. El valor predeterminado es `true`.

**Devuelve**

El `schema_merge` complemento devuelve el simiar de salida a lo que devuelve el operador [getschema.](./getschemaoperator.md)

**Ejemplos**

Combinar con un esquema que tenga una nueva columna anexada:

```kusto
let schema1 = datatable(Uri:string, HttpStatus:int)[] | getschema;
let schema2 = datatable(Uri:string, HttpStatus:int, Referrer:string)[] | getschema;
union schema1, schema2 | evaluate schema_merge()
```

*Resultado*

|ColumnName | ColumnOrdinal | DataType | ColumnType|
|---|---|---|---|
|Identificador URI|0|System.String|string|
|HttpStatus|1|System.Int32|int|
|Referrer|2|System.String|string|

Combinar con un esquema que`HttpStatus` tiene diferentes `1` `2` tipos de columna (cambios ordinales de a la nueva variante):

```kusto
let schema1 = datatable(Uri:string, HttpStatus:int)[] | getschema;
let schema2 = datatable(Uri:string, Referrer:string, HttpStatus:int)[] | getschema;
union schema1, schema2 | evaluate schema_merge()
```

*Resultado*

|ColumnName | ColumnOrdinal | DataType | ColumnType|
|---|---|---|---|
|Identificador URI|0|System.String|string|
|Referrer|1|System.String|string|
|HttpStatus|-1|ERROR(unknown CSL type:ERROR(columns are out of order))|ERROR(las columnas están fuera de servicio)|

Combinar con un esquema que tiene `PreserveOrder` un `false` orden de columnas diferente, pero con establecido en este momento:

```kusto
let schema1 = datatable(Uri:string, HttpStatus:int)[] | getschema;
let schema2 = datatable(Uri:string, Referrer:string, HttpStatus:int)[] | getschema;
union schema1, schema2 | evaluate schema_merge(PreserveOrder = false)
```

*Resultado*

|ColumnName | ColumnOrdinal | DataType | ColumnType|
|---|---|---|---|
|Identificador URI|0|System.String|string
|Referrer|1|System.String|string
|HttpStatus|2|System.Int32|int|