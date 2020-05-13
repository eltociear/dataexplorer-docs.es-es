---
title: 'series_divide (): Explorador de datos de Azure'
description: En este artículo se describe series_divide () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 7d5bdba030687c17c355eb72ce2fc9c358c10ebd
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372844"
---
# <a name="series_divide"></a>series_divide()

Calcula la división de elementos de dos entradas de serie numérica.

**Sintaxis**

`series_divide(`*Series1* `,` *series2*`)`

**Argumentos**

* *Series1, series2*: matrices numéricas de entrada, la primera que se va a asignar a elementos dividida por la segunda en un resultado de matriz dinámica. Todos los argumentos deben ser matrices dinámicas. 

**Devuelve**

Matriz dinámica de la operación de división en elementos calculada entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices de distintos tamaños) produce un `null` valor de elemento.

Nota: la serie de resultados es de tipo Double, aunque las entradas sean números enteros. La división por cero sigue la división Double por cero (por ejemplo, 2/0 produce Double (+ INF)).

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| project s1 = pack_array(x,y,z), s2 = pack_array(z, y, x)
| extend s1_divide_s2 = series_divide(s1, s2)
```

|s1         |s2|        s1_divide_s2|
|---|---|---|
|[1, 2, 4]    |[4, 2, 1]|   [0.25, 1.0, 4.0]|
|[2, 4, 8]    |[8, 4, 2]|   [0.25, 1.0, 4.0]|
|[3, 6, 12]   |[12, 6, 3]|  [0.25, 1.0, 4.0]|
