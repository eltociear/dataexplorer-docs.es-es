---
title: 'series_seasonal (): Explorador de datos de Azure'
description: En este artículo se describe series_seasonal () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 88160a55ba8342e3ed6bce90ec77c5a370ab7358
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372469"
---
# <a name="series_seasonal"></a>series_seasonal()

Calcula el componente estacional de una serie según el período de temporada detectado o determinado.

**Sintaxis**

`series_seasonal(`*serie* `[,` de *período* de`])`

**Argumentos**

* *series*: matriz dinámica numérica de entrada
* *period* (opcional): número entero de depósitos en cada período estacional, valores posibles:
    *  -1 (valor predeterminado): detectar automáticamente el período con [series_periods_detect ()](series-periods-detectfunction.md) con un umbral de *0,7*, devuelve ceros si no se detecta la estacionalidad.
    * entero positivo: se usará como el período del componente estacional.
    * cualquier otro valor: omitir la estacionalidad y devolver una serie de ceros

**Devuelve**

Matriz dinámica de la misma longitud que la entrada de la *serie* que contiene el componente estacional calculado de la serie. El componente estacional se calcula como la *mediana* de todos los valores correspondientes a la ubicación del hueco en los períodos.

**Vea también:**

* [series_periods_detect()](series-periods-detectfunction.md)
* [series_periods_validate()](series-periods-validatefunction.md)

**Ejemplos**

**1. detectar automáticamente el período**

En el ejemplo siguiente, el período de la serie se detecta automáticamente, se detecta que el período de la primera serie es de 6 bandejas y los de la segunda, el período de la tercera serie es demasiado corto para ser detectado y devuelve una serie de ceros (consulte el ejemplo siguiente sobre cómo forzar el período).

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print s=dynamic([2,5,3,4,3,2,1,2,3,4,3,2,1,2,3,4,3,2,1,2,3,4,3,2,1])
| union (print s=dynamic([8,12,14,12,10,10,12,14,12,10,10,12,14,12,10,10,12,14,12,10]))
| union (print s=dynamic([1,3,5,2,4,6,1,3,5,2,4,6]))
| extend s_seasonal = series_seasonal(s)
```

|s|s_seasonal|
|---|---|
|[2, 5, 3, 4, 3, 2, 1, 2, 3, 4, 3, 2, 1, 2, 3, 4, 3, 2, 1, 2, 3, 4, 3, 2, 1]|[1.0, 2.0, 3.0, 4.0, 3.0, 2.0, 1.0, 2.0, 3.0, 4.0, 3.0, 2.0, 1.0, 2.0, 3.0, 4.0, 3.0, 2.0, 1.0, 2.0, 3.0, 4.0, 3.0, 2.0, 1.0]|
|[8, 12, 14, 12, 10, 10, 12, 14, 12, 10, 10, 12, 14, 12, 10, 10, 12, 14, 12, 10]|[10.0, 12.0, 14.0, 12.0, 10.0, 10.0, 12.0, 14.0, 12.0, 10.0, 10.0, 12.0, 14.0, 12.0, 10.0, 10.0, 12.0, 14.0, 12.0, 10.0]|
|[1, 3, 5, 2, 4, 6, 1, 3, 5, 2, 4, 6]|[0.0, 0,0, 0,0, 0,0, 0,0, 0,0, 0,0, 0.0, 0.0, 0.0, 0.0, 0.0]|



**2. forzar un período**

En el ejemplo siguiente, el período de la serie es demasiado corto para ser detectado por [series_periods_detect ()](series-periods-detectfunction.md) , por lo que se fuerza explícitamente el período para obtener el patrón estacional.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print s=dynamic([1,3,5,1,3,5,2,4,6]) 
| union (print s=dynamic([1,3,5,2,4,6,1,3,5,2,4,6]))
| extend s_seasonal = series_seasonal(s,3)
```

|s|s_seasonal|
|---|---|
|[1, 3, 5, 1, 3, 5, 2, 4, 6]|[1.0, 3.0, 5.0, 1.0, 3.0, 5.0, 1.0, 3.0, 5.0]|
|[1, 3, 5, 2, 4, 6, 1, 3, 5, 2, 4, 6]|[1.5, 3.5, 5.5, 1.5, 3.5, 5.5, 1.5, 3.5, 5.5, 1.5, 3.5, 5.5]|
