---
title: series_seasonal() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe series_seasonal() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 559731ab7dca2e49e124a856ca7a66a912583b62
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663127"
---
# <a name="series_seasonal"></a>series_seasonal()

Calcula el componente estacional de una serie según el período estacional detectado o dado.

**Sintaxis**

`series_seasonal(`*período de la* *serie* `[,``])`

**Argumentos**

* *serie*: Matriz dinámica numérica de entrada
* *período* (opcional): número entero de ubicaciones en cada período estacional, valores posibles:
    *  -1 (predeterminado): detecta automáticamente el período utilizando [series_periods_detect()](series-periods-detectfunction.md) con un umbral de *0,7*, devuelve ceros si no se detecta estacionalidad
    * entero positivo: se utilizará como período para el componente estacional
    * cualquier otro valor: ignorar la estacionalidad y devolver una serie de ceros

**Devuelve**

Matriz dinámica de la misma longitud que la entrada de *serie* que contiene el componente estacional calculado de la serie. El componente estacional se calcula como la *mediana* de todos los valores correspondientes a la ubicación de la ubicación de la ubicación a lo largo de los períodos.

**Véase también:**

* [series_periods_detect()](series-periods-detectfunction.md)
* [series_periods_validate()](series-periods-validatefunction.md)

**Ejemplos**

**1. Detección automática del período**

En el ejemplo siguiente, el período de la serie se detecta automáticamente, se detecta que el período de la primera serie es de 6 bins y los segundos 5 bins, el período de la tercera serie es demasiado corto para ser detectado y devuelve una serie de ceros (véase el siguiente ejemplo sobre cómo forzar el período).

```kusto
print s=dynamic([2,5,3,4,3,2,1,2,3,4,3,2,1,2,3,4,3,2,1,2,3,4,3,2,1])
| union (print s=dynamic([8,12,14,12,10,10,12,14,12,10,10,12,14,12,10,10,12,14,12,10]))
| union (print s=dynamic([1,3,5,2,4,6,1,3,5,2,4,6]))
| extend s_seasonal = series_seasonal(s)
```

|s|s_seasonal|
|---|---|
|[2,5,3,4,3,2,1,2,3,4,3,2,1,2,3,4,3,2,1,2,3,4,3,2,1]|[1.0,2.0,3.0,4.0,3.0,2.0,1.0,2.0,3.0,4.0,3.0,2.0,1.0,2.0,3.0,4.0,3.0,2.0,1.0,2.0,3.0,4.0,3.0,2.0,1.0]|
|[8,12,14,12,10,10,12,14,12,10,10,12,14,12,10,10,12,14,12,10]|[10.0,12.0,14.0,12.0,10.0,10.0,12.0,14.0,12.0,10.0,10.0,12.0,14.0,12.0,10.0,10.0,12.0,14.0,12.0,10.0]|
|[1,3,5,2,4,6,1,3,5,2,4,6]|[0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0,0.0]|



**2. Forzar un período**

En el siguiente ejemplo, el período de la serie es demasiado corto para ser detectado por [series_periods_detect(),](series-periods-detectfunction.md) por lo que forzamos el período explícitamente para obtener el patrón estacional.

```kusto
print s=dynamic([1,3,5,1,3,5,2,4,6]) 
| union (print s=dynamic([1,3,5,2,4,6,1,3,5,2,4,6]))
| extend s_seasonal = series_seasonal(s,3)
```

|s|s_seasonal|
|---|---|
|[1,3,5,1,3,5,2,4,6]|[1.0,3.0,5.0,1.0,3.0,5.0,1.0,3.0,5.0]|
|[1,3,5,2,4,6,1,3,5,2,4,6]|[1.5,3.5,5.5,1.5,3.5,5.5,1.5,3.5,5.5,1.5,3.5,5.5]|
