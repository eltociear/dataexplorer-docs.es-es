---
title: 'Operador de ejemplo: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador de ejemplo en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/18/2020
ms.openlocfilehash: 757830bde0c56ac727d5240c01ca4768eab28877
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510037"
---
# <a name="sample-operator"></a>Operador sample

Devuelve hasta el número especificado de filas aleatorias de la tabla de entrada.

```kusto
T | sample 5
```

**Sintaxis**

_T_ `| sample` _NumberOfRows_

**Argumentos**

- _NumberOfRows_: el número de filas de _T_ que se van a devolver. Puede especificar cualquier expresión numérica.

**Notas**

- `sample`se prepara para la velocidad en lugar de una distribución uniforme de los valores. Específicamente, significa que no producirá resultados "justos" si se utiliza después de `union` los `join` operadores que unen 2 conjuntos de datos de diferentes tamaños (como un u operadores). Se recomienda usar `sample` justo después de la referencia de la tabla y los filtros.

- `sample`es un operador no determinista y devolverá un conjunto de resultados diferente cada vez que se evalúe durante la consulta. Por ejemplo, la siguiente consulta produce dos filas diferentes (incluso si se espera que una de ellas devuelva la misma fila dos veces).

```kusto
let _data = range x from 1 to 100 step 1;
let _sample = _data | sample 1;
union (_sample), (_sample)
```

| x   |
| --- |
| 83  |
| 3   |

Para asegurarse de que `_sample` en el ejemplo anterior se calcula una vez, se puede utilizar la función [materialize():](./materializefunction.md)

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

- si desea muestrear un determinado porcentaje de los datos (en lugar de un número especificado de filas), puede utilizar

```kusto
StormEvents | where rand() < 0.1
```

- Si desea muestrear claves en lugar de filas (por ejemplo, ejemplo 10 identificadores [`sample-distinct`](./sampledistinctoperator.md) y obtener `in` todas las filas para estos identificadores) puede usar en combinación con el operador.

**Ejemplos**

```kusto
let sampleEpisodes = StormEvents | sample-distinct 10 of EpisodeId;
StormEvents
| where EpisodeId in (sampleEpisodes)
```