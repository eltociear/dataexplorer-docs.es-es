---
title: 'operador de ejemplo: Azure Explorador de datos'
description: En este artículo se describe el operador de ejemplo en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/18/2020
ms.openlocfilehash: 4915371127acd229845cc9eac1ea1400484c313f
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372979"
---
# <a name="sample-operator"></a>Operador sample

Devuelve hasta el número especificado de filas aleatorias de la tabla de entrada.

```kusto
T | sample 5
```

**Sintaxis**

_T_ `| sample` _NumberOfRows_

**Argumentos**

- _NumberOfRows_: el número de filas de _T_ que se va a devolver. Puede especificar cualquier expresión numérica.

**Notas**

- `sample`está orientado a la velocidad en lugar de a la distribución de valores. En concreto, esto significa que no producirá resultados "equitativos" si se usan después de operadores que tienen conjuntos de datos Union 2 de distintos tamaños (como `union` `join` operadores o). Se recomienda usar `sample` justo después de la referencia de tabla y los filtros.

- `sample`es un operador no determinista y devolverá un conjunto de resultados diferente cada vez que se evalúe durante la consulta. Por ejemplo, la consulta siguiente produce dos filas diferentes (incluso si una esperaría devolver la misma fila dos veces).

```kusto
let _data = range x from 1 to 100 step 1;
let _sample = _data | sample 1;
union (_sample), (_sample)
```

| x   |
| --- |
| 83  |
| 3   |

Para asegurarse de que, en el ejemplo anterior `_sample` , se calcula una vez, se puede usar la función [materializar ()](./materializefunction.md) :

```kusto
let _data = range x from 1 to 100 step 1;
let _sample = materialize(_data | sample 1);
union (_sample), (_sample)
```

| x   |
| --- |
| 34  |
| 34  |

**Sugerencias**

- Si desea que muestre un determinado porcentaje de los datos (en lugar de un número especificado de filas), puede usar

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents | where rand() < 0.1
```

- Si desea muestrear claves en lugar de filas (por ejemplo, identificadores de ejemplo 10 y obtener todas las filas de estos identificadores), puede usar [`sample-distinct`](./sampledistinctoperator.md) en combinación con el `in` operador.

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
let sampleEpisodes = StormEvents | sample-distinct 10 of EpisodeId;
StormEvents
| where EpisodeId in (sampleEpisodes)
```
