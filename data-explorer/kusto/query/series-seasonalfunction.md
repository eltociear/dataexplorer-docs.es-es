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
ms.openlocfilehash: 054d4be758001609fbc3100a4a6c8698ef8f69f6
ms.sourcegitcommit: ae72164adc1dc8d91ef326e757376a96ee1b588d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2020
ms.locfileid: "84717315"
---
# <a name="series_seasonal"></a>series_seasonal()

Calcula el componente estacional de una serie, según el período de temporada detectado o determinado.

**Sintaxis**

`series_seasonal(`*serie* `[,` de *período* de`])`

**Argumentos**

* *series*: matriz dinámica numérica de entrada
* *period* (opcional): número entero de depósitos en cada período estacional, valores posibles:
    *  -1 (valor predeterminado): detecta automáticamente el período mediante [series_periods_detect ()](series-periods-detectfunction.md) con un umbral de *0,7*. Devuelve ceros si no se detecta estacionalidad
    * Entero positivo: se usa como el período del componente estacional
    * Cualquier otro valor: omite la estacionalidad y devuelve una serie de ceros.

**Devuelve**

Matriz dinámica de la misma longitud que la entrada de la *serie* que contiene el componente estacional calculado de la serie. El componente estacional se calcula como la *mediana* de todos los valores que corresponden a la ubicación de la ubicación, en los períodos.

## <a name="examples"></a>Ejemplos

### <a name="auto-detect-the-period"></a>Detección automática del período

En el ejemplo siguiente, el período de la serie se detecta automáticamente. El período de la primera serie se detecta en seis ubicaciones y en las cinco. El período de la tercera serie es demasiado corto para detectarse y devuelve una serie de ceros. Vea el ejemplo siguiente sobre [Cómo forzar el período](#force-a-period).

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

### <a name="force-a-period"></a>Forzar un período

En este ejemplo, el período de la serie es demasiado corto para ser detectado por [series_periods_detect ()](series-periods-detectfunction.md), por lo que se fuerza explícitamente el período para obtener el patrón estacional.

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
 
## <a name="next-steps"></a>Pasos siguientes

* [series_periods_detect()](series-periods-detectfunction.md)
* [series_periods_validate()](series-periods-validatefunction.md)
