---
title: como operador- Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe como operador en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 05dc96fb7eec773d1e55d8b94a33cdda928622ff
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518435"
---
# <a name="as-operator"></a>Operador as

Enlaza un nombre a la expresión tabular de entrada del operador, lo que permite que la consulta haga referencia al valor de la expresión tabular varias veces sin interrumpir la consulta y enlazar un nombre a través de la [instrucción let](letstatement.md).

**Sintaxis**

*T* `|` T `as` `hint.materialized` [ `=` ] *Nombre* `true`

**Argumentos**

* *T*: Una expresión tabular.
* *Nombre*: Un nombre temporal para la expresión tabular.
* `hint.materialized`: Si `true`se establece en , el valor de la expresión tabular se materializará como si estuviera envuelta por una llamada a la función [materialize().](./materializefunction.md)

**Notas**

* El nombre `as` dado por se `withsource=` utilizará en `source_` la columna de [unión](./unionoperator.md), la columna de [búsqueda](./findoperator.md), y la `$table` columna de [búsqueda](./searchoperator.md).

* La expresión tabular nombrada mediante el operador en`$left`una entrada tabular externa de [combinación](./joinoperator.md)(`$right`) también se puede utilizar en la entrada interna tabular de la unión ( ).

**Ejemplos**

```kusto
// 1. In the following 2 example the union's generated TableName column will consist of 'T1' and 'T2'
range x from 1 to 10 step 1 
| as T1 
| union withsource=TableName T2

union withsource=TableName (range x from 1 to 10 step 1 | as T1), T2

// 2. In the following example, the 'left side' of the join will be: 
//      MyLogTable filtered by type == "Event" and Name == "Start"
//    and the 'right side' of the join will be: 
//      MyLogTable filtered by type == "Event" and Name == "Stop"
MyLogTable  
| where type == "Event"
| as T
| where Name == "Start"
| join (
    T
    | where Name == "Stop"
) on ActivityId
```