---
title: table() (función de ámbito) - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe table() (función de ámbito) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 233baebaf51cdc8b07cdd32cec9bd10a54695c1c
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81766158"
---
# <a name="table-scope-function"></a>table() (función de ámbito)

La función table() hace referencia a una `string`tabla proporcionando su nombre como una expresión de tipo .

```kusto
table('StormEvent')
```

**Sintaxis**

`table``(` *NombreTabla* `,` [ *DataScope*]`)`

**Argumentos**

::: zone pivot="azuredataexplorer"

* *TableName*: una `string` expresión de tipo que proporciona el nombre de la tabla a la que se hace referencia. El valor de esta expresión debe ser constante en el punto de llamada a la función (es decir, no puede variar según el contexto de datos).

* *DataScope*: Un parámetro `string` opcional de tipo que se puede utilizar para restringir la referencia de tabla a los datos de acuerdo con cómo estos datos están dentro de la política de [caché](../management/cachepolicy.md)efectiva de la tabla. Si se utiliza, el argumento `string` real debe ser una expresión constante que tenga uno de los siguientes valores posibles:

    - `"hotcache"`: Solo se hará referencia a los datos que se clasifican como caché en caliente.
    - `"all"`: se hará referencia a todos los datos de la tabla.
    - `"default"`: es lo `"all"`mismo que , excepto si `"hotcache"` el clúster se ha configurado para usar como predeterminado por el administrador del clúster.

::: zone-end

::: zone pivot="azuremonitor"

* *TableName*: una `string` expresión de tipo que proporciona el nombre de la tabla a la que se hace referencia. El valor de esta expresión debe ser constante en el punto de llamada a la función (es decir, no puede variar según el contexto de datos).

* *DataScope*: un parámetro `string` opcional de tipo que se puede utilizar para restringir la referencia de tabla a los datos de acuerdo con cómo estos datos están dentro de la directiva de caché efectiva de la tabla. Si se utiliza, el argumento `string` real debe ser una expresión constante que tenga uno de los siguientes valores posibles:

    - `"hotcache"`: Solo se hará referencia a los datos que se clasifican como caché en caliente.
    - `"all"`: se hará referencia a todos los datos de la tabla.
    - `"default"`: Esto es `"all"`lo mismo que .

::: zone-end

## <a name="examples"></a>Ejemplos

### <a name="use-table-to-access-table-of-the-current-database"></a>Utilice table() para acceder a la tabla de la base de datos actual

```kusto
table('StormEvent') | count
```

|Count|
|---|
|59066|

### <a name="use-table-inside-let-statements"></a>Use table() dentro de las instrucciones let

La misma consulta que la anterior se puede reescribir para utilizar `tableName` la función en línea (instrucción let) que recibe un parámetro - que se pasa a la función table().

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

### <a name="use-table-inside-functions"></a>Usar table() dentro de Funciones

La misma consulta que la anterior se puede reescribir para `tableName` usarla en una función que recibe un parámetro, que se pasa a la función table().

```kusto
.create function foo(tableName:string)
{
    table(tableName) | count
};
```

::: zone pivot="azuredataexplorer"

Nota: estas funciones solo se pueden utilizar localmente y no en la consulta entre **clústeres.**

::: zone-end

### <a name="use-table-with-non-constant-parameter"></a>Utilice table() con parámetro no constante

Un parámetro, que no es una cadena constante escalar, `table()` no se puede pasar como parámetro para la función.

A continuación, se muestra un ejemplo de solución alternativa para tal caso.

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
