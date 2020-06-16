---
title: 'complemento de schema_merge: Azure Explorador de datos'
description: En este artículo se describe schema_merge complemento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/16/2020
ms.openlocfilehash: b1f3ef10ac5cee3eb9bc1c1dca4c0de26bd85477
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84780208"
---
# <a name="schema_merge-plugin"></a>complemento de schema_merge

Combina definiciones de esquema tabular en esquema unificado. 

Se espera que las definiciones de esquema estén en el formato generado por el [`getschema`](./getschemaoperator.md) operador.

La `schema merge` operación combina columnas en los esquemas de entrada e intenta reducir los tipos de datos a los comunes. Si no se pueden reducir los tipos de datos, se muestra un error en la columna problemático.

```kusto
let Schema1=Table1 | getschema;
let Schema2=Table2 | getschema;
union Schema1, Schema2 | evaluate schema_merge()
```

**Sintaxis**

`T``|` `evaluate` `schema_merge(` *PreserveOrder*`)`

**Argumentos**

* *PreserveOrder*: (opcional) cuando se establece en `true` , indica al complemento que valide el orden de las columnas según lo definido por el primer esquema tabular que se mantiene. Si la misma columna está en varios esquemas, el ordinal de la columna debe ser como el ordinal de columna del primer esquema en el que apareció. El valor predeterminado es `true`.

**Devuelve**

El `schema_merge` complemento devuelve una salida similar a la que [`getschema`](./getschemaoperator.md) devuelve el operador.

**Ejemplos**

Combinar con un esquema que tiene anexada una nueva columna.

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

Combinar con un esquema que tiene un orden de columnas diferente ( `HttpStatus` los cambios ordinales de `1` a `2` en la nueva variante).

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
|HttpStatus|-1|ERROR (tipo de CSL desconocido: ERROR (las columnas no están ordenadas))|ERROR (las columnas no están ordenadas)|

Combine con un esquema que tenga diferente orden de columnas, pero con `PreserveOrder` establecido en `false` .

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
