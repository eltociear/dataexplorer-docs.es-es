---
title: row_cumsum() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe row_cumsum() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 92ebec75dcd7e44d59f964dc735e857f22f1ad00
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510224"
---
# <a name="row_cumsum"></a>row_cumsum()

Calcula la suma acumulada de una columna en un conjunto de [filas serializado.](./windowsfunctions.md#serialized-row-set)

**Sintaxis**

`row_cumsum``(` *Término* `,` [ *Reiniciar*]`)`

* *Término* es una expresión que indica el valor que se va a sumar.
  La expresión debe ser un escalar de uno `decimal` `int`de `long`los `real`siguientes tipos: , , , o . Los valores *de Null Term* no afectan a la suma.
* *Reiniciar* es un argumento `bool` opcional de tipo que indica cuándo se debe reiniciar la operación de acumulación (establecer de nuevo en 0). Se puede utilizar para indicar particiones de los datos; ver el segundo ejemplo a continuación.

**Devuelve**

La función devuelve la suma acumulada de su argumento.

**Ejemplos**

En el ejemplo siguiente se muestra cómo calcular la suma acumulada de los primeros enteros par.

```kusto
datatable (a:long) [
    1, 2, 3, 4, 5, 6, 7, 8, 9, 10
]
| where a%2==0
| serialize cs=row_cumsum(a)
```

a    | cs
-----|-----
2    | 2
4    | 6
6    | 12
8    | 20
10   | 30

Este ejemplo muestra cómo calcular la suma `salary`acumulada (aquí, de ) `name`cuando los datos se particionan (aquí, por ):

```kusto
datatable (name:string, month:int, salary:long)
[
    "Alice", 1, 1000,
    "Bob",   1, 1000,
    "Alice", 2, 2000,
    "Bob",   2, 1950,
    "Alice", 3, 1400,
    "Bob",   3, 1450,
]
| order by name asc, month asc
| extend total=row_cumsum(salary, name != prev(name))
```

name   | month  | Salario  | total
-------|--------|---------|------
Alice  | 1      | 1000    | 1000
Alice  | 2      | 2000    | 3000
Alice  | 3      | 1400    | 4400
Bob    | 1      | 1000    | 1000
Bob    | 2      | 1950    | 2950
Bob    | 3      | 1450    | 4400