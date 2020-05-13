---
title: 'series_subtract (): Explorador de datos de Azure'
description: En este artículo se describe series_subtract () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 388f24f12993bcdc91d86bfc3f3f20967e0b1cc5
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372415"
---
# <a name="series_subtract"></a>series_subtract()

Calcula la sustracción de elementos de dos entradas de serie numérica.

**Sintaxis**

`series_subtract(`*Series1* `,` *series2*`)`

**Argumentos**

* *Series1, series2*: matrices numéricas de entrada, la segunda que se va a restar del primer elemento en un resultado de matriz dinámica. Todos los argumentos deben ser matrices dinámicas. 

**Devuelve**

Matriz dinámica de la operación de resta calculada de elementos calculados entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices de distintos tamaños) produce un `null` valor de elemento.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| project s1 = pack_array(x,y,z), s2 = pack_array(z, y, x)
| extend s1_subtract_s2 = series_subtract(s1, s2)
```

|s1|s2|s1_subtract_s2|
|---|---|---|
|[1, 2, 4]|[4, 2, 1]|[-3, 0,3]|
|[2, 4, 8]|[8, 4, 2]|[-6, 0, 6]|
|[3, 6, 12]|[12, 6, 3]|[-9, 0, 9]|
