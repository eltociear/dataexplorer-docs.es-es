---
title: 'series_multiply (): Explorador de datos de Azure'
description: En este artículo se describe series_multiply () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 1f88cdfc1490f8b00d8104286441e366aaf46f3f
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372574"
---
# <a name="series_multiply"></a>series_multiply()

Calcula la multiplicación por elemento de dos entradas de serie numéricas.

**Sintaxis**

`series_multiply(`*Series1* `,` *series2*`)`

**Argumentos**

* *Series1, series2*: matrices numéricas de entrada, que se multiplicarán por elementos en un resultado de matriz dinámica. Todos los argumentos deben ser matrices dinámicas. 

**Devuelve**

Matriz dinámica de la operación de multiplicación de elementos calculados entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices de distintos tamaños) produce un `null` valor de elemento.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| project s1 = pack_array(x,y,z), s2 = pack_array(z, y, x)
| extend s1_multiply_s2 = series_multiply(s1, s2)
```

|s1         |s2|        s1_multiply_s2|
|---|---|---|
|[1, 2, 4]    |[4, 2, 1]|   [4, 4, 4]|
|[2, 4, 8]    |[8, 4, 2]|   [16, 16, 16]|
|[3, 6, 12]   |[12, 6, 3]|  [36, 36, 36]|
