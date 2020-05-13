---
title: 'Table () (función de ámbito): Azure Explorador de datos'
description: En este artículo se describe Table () (función SCOPE) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 0d5f44d621612e90d83a93f2f5831630520d4ba0
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83371749"
---
# <a name="table-scope-function"></a>Table () (función SCOPE)

La función Table () hace referencia a una tabla proporcionando su nombre como una expresión de tipo `string` .

```kusto
table('StormEvent')
```

**Sintaxis**

`table``(` *TableName* [pág `,` *DataScope*]`)`

**Argumentos**

::: zone pivot="azuredataexplorer"

* *TableName*: una expresión de tipo `string` que proporciona el nombre de la tabla a la que se hace referencia. El valor de esta expresión debe ser constante en el punto de la llamada a la función (es decir, no puede variar según el contexto de datos).

* MyPolicy *: un*parámetro opcional de tipo `string` que se puede usar para restringir la referencia de tabla a los datos según el modo en que estos datos caen en la [Directiva de caché](../management/cachepolicy.md)efectiva de la tabla. Si se usa, el argumento real debe ser una `string` expresión constante con uno de los siguientes valores posibles:

    - `"hotcache"`: Solo se hará referencia a los datos que se clasifican como caché de acceso frecuente.
    - `"all"`: Se hará referencia a todos los datos de la tabla.
    - `"default"`: Es igual que `"all"` , excepto si el clúster se ha configurado para que lo use el `"hotcache"` Administrador del clúster como valor predeterminado.

::: zone-end

::: zone pivot="azuremonitor"

* *TableName*: una expresión de tipo `string` que proporciona el nombre de la tabla a la que se hace referencia. El valor de esta expresión debe ser constante en el punto de la llamada a la función (es decir, no puede variar según el contexto de datos).

* MyPolicy *: un*parámetro opcional de tipo `string` que se puede usar para restringir la referencia de tabla a los datos según el modo en que estos datos caen en la Directiva de caché efectiva de la tabla. Si se usa, el argumento real debe ser una `string` expresión constante con uno de los siguientes valores posibles:

    - `"hotcache"`: Solo se hará referencia a los datos que se clasifican como caché de acceso frecuente.
    - `"all"`: Se hará referencia a todos los datos de la tabla.
    - `"default"`: Es igual que `"all"` .

::: zone-end

## <a name="examples"></a>Ejemplos

### <a name="use-table-to-access-table-of-the-current-database"></a>Usar Table () para tener acceso a la tabla de la base de datos actual

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
table('StormEvent') | count
```

|Count|
|---|
|59066|

### <a name="use-table-inside-let-statements"></a>Usar Table () dentro de instrucciones Let

La misma consulta que antes se puede volver a escribir para usar la función inline (instrucción Let) que recibe un parámetro `tableName` , que se pasa a la función Table ().

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
let foo = (tableName:string)
{
    table(tableName) | count
};
foo('help')
```

|Count|
|---|
|59066|

### <a name="use-table-inside-functions"></a>Usar Table () dentro de las funciones

Se puede volver a escribir la misma consulta anterior para usarla en una función que recibe un parámetro `tableName` , que se pasa a la función Table ().

```kusto
.create function foo(tableName:string)
{
    table(tableName) | count
};
```

::: zone pivot="azuredataexplorer"

**Nota:** estas funciones solo se pueden usar de forma local y no en la consulta entre clústeres.

::: zone-end

### <a name="use-table-with-non-constant-parameter"></a>Usar Table () con un parámetro no constante

Un parámetro, que no es una cadena de constante escalar, no se puede pasar como parámetro a la `table()` función.

A continuación, se proporciona un ejemplo de solución alternativa para este caso.

```kusto
let T1 = print x=1;
let T2 = print x=2;
let _choose = (_selector:string)
{
    union
    (T1 | where _selector == 'T1'),
    (T2 | where _selector == 'T2')
};
_choose('T2')

```

|x|
|---|
|2|
