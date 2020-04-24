---
title: row_number() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe row_number() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: c8cb01ed098d24632154215ddf06dc2ab1d72695
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510173"
---
# <a name="row_number"></a>row_number()

Devuelve el índice de la fila actual en un conjunto de [filas serializado.](./windowsfunctions.md#serialized-row-set)
El índice de fila `1` comienza de forma predeterminada en `1` la primera fila y se incrementa por para cada fila adicional.
Opcionalmente, el índice de fila puede `1`comenzar con un valor diferente que .
Además, el índice de fila se puede restablecer según algunos predicados proporcionados.

**Sintaxis**

`row_number``(` [*StartingIndex* [`,` *Reiniciar*]]`)`

* *StartingIndex* es una expresión `long` constante de tipo que indica el valor del índice de fila para iniciar en (o para reiniciar en). El valor predeterminado es `1`.
* *Reiniciar* es un argumento `bool` opcional de tipo que indica cuándo se va a reiniciar la numeración en el valor *StartingIndex.* Si no se proporciona, `false` se utiliza el valor predeterminado de.

**Devuelve**

La función devuelve el índice de fila `long`de la fila actual como un valor de tipo .

**Ejemplos**

En el ejemplo siguiente se devuelve una`a`tabla con `10` dos `1`columnas, la primera`rn`columna ( `1` ) `10`con números de abajo a , y la segunda columna ( ) con números de hasta:

```kusto
range a from 1 to 10 step 1
| sort by a desc
| extend rn=row_number()
```

El siguiente ejemplo es similar al anterior,`rn`solo `7`la segunda columna ( ) comienza en:

```kusto
range a from 1 to 10 step 1
| sort by a desc
| extend rn=row_number(7)
```

En el último ejemplo se muestra cómo se pueden particionar los datos y numerar las filas por cada partición. Aquí, particionamos `Airport`los datos por:

```kusto
datatable (Airport:string, Airline:string, Departures:long)
[
  "TLV", "LH", 1,
  "TLV", "LY", 100,
  "SEA", "LH", 1,
  "SEA", "BA", 2,
  "SEA", "LY", 0
]
| sort by Airport asc, Departures desc
| extend Rank=row_number(1, prev(Airport) != Airport)
```

La ejecución de esta consulta produce el siguiente resultado:

Aeropuerto  | Línea aérea  | Salidas  | Rank
---------|----------|-------------|------
SEA      | BA       | 2           | 1
SEA      | Lh       | 1           | 2
SEA      | LY       | 0           | 3
Tlv      | LY       | 100         | 1
Tlv      | Lh       | 1           | 2