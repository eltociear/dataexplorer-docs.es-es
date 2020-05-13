---
title: 'set_intersect (): Explorador de datos de Azure'
description: En este artículo se describe set_intersect () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/02/2019
ms.openlocfilehash: 23b751dce38f5b595ba081c9a29e1b1a5442c96f
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372352"
---
# <a name="set_intersect"></a>set_intersect()

Devuelve una `dynamic` matriz (JSON) del conjunto de todos los valores distintos que se encuentran en todas las matrices: (arr1 ∩ arr2 ∩...).

**Sintaxis**

`set_intersect(`*arr1* `, ` *arr2* `[` ,` *arr3*, ...])`

**Argumentos**

* *arr1... arrN*: matrices de entrada para crear un conjunto de intersección (al menos dos matrices). Todos los argumentos deben ser matrices dinámicas (vea [pack_array](packarrayfunction.md)). 

**Devuelve**

Devuelve una matriz dinámica del conjunto de todos los valores distintos que se encuentran en todas las matrices. Vea [`set_union()`](setunionfunction.md) y [`set_difference()`](setdifferencefunction.md) .

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| extend w = z * 2
| extend a1 = pack_array(x,y,x,z), a2 = pack_array(x, y), a3 = pack_array(w,x)
| project set_intersect(a1, a2, a3)
```

|Column1|
|---|
|[1]|
|dos|
|[3]|

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print arr = set_intersect(dynamic([1, 2, 3]), dynamic([4,5]))
```

|ARR|
|---|
|[]|
