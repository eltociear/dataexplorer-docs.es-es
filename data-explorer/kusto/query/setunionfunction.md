---
title: 'set_union (): Explorador de datos de Azure'
description: En este artículo se describe set_union () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/02/2019
ms.openlocfilehash: 7e719ffa448ce3060a0a520de2c031b7d4b7d109
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372335"
---
# <a name="set_union"></a>set_union()

Devuelve una `dynamic` matriz (JSON) del conjunto de todos los valores distintos que se encuentran en cualquiera de las matrices: (arr1 ∪ arr2 ∪...).

**Sintaxis**

`set_union(`*arr1* `, ` *arr2* `[` ,` *arr3*, ...]``)`

**Argumentos**

* *arr1... arrN*: matrices de entrada para crear un conjunto de uniones (al menos dos matrices). Todos los argumentos deben ser matrices dinámicas (vea [pack_array](packarrayfunction.md)). 

**Devuelve**

Devuelve una matriz dinámica del conjunto de todos los valores distintos que se encuentran en cualquiera de las matrices. Vea [`set_intersect()`](setintersectfunction.md) y [`set_difference()`](setdifferencefunction.md) .

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| extend w = z * 2
| extend a1 = pack_array(x,y,x,z), a2 = pack_array(x, y), a3 = pack_array(w)
| project set_union(a1, a2, a3)
```

|Column1|
|---|
|[1, 2, 4, 8]|
|[2, 4, 8, 16]|
|[3, 6, 12, 24]|
