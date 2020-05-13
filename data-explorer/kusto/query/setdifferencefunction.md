---
title: set_difference ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe set_difference () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/02/2019
ms.openlocfilehash: 7e13a9b652e1bdadb325cd866bddd78761b25b85
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372395"
---
# <a name="set_difference"></a>set_difference()

Devuelve una `dynamic` matriz (JSON) del conjunto de todos los valores distintos que se encuentran en la primera matriz pero que no están en otras matrices (((arr1 \ arr2) \ arr3) \...).

**Sintaxis**

`set_difference(`*arr1* `, ` *arr2* `[` ,` *arr3*, ...])`

**Argumentos**

* *arr1... arrN*: matrices de entrada para crear un conjunto de diferencias (al menos dos matrices). Todos los argumentos deben ser matrices dinámicas (vea [pack_array](packarrayfunction.md)). 

**Devuelve**

Devuelve una matriz dinámica del conjunto de todos los valores distintos que se encuentran en arr1 pero que no están en otras matrices. Vea [`set_union()`](setunionfunction.md) y [`set_intersect()`](setintersectfunction.md) .

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| extend w = z * 2
| extend a1 = pack_array(x,y,x,z), a2 = pack_array(x, y), a3 = pack_array(x,y,w)
| project set_difference(a1, a2, a3)
```

|Column1|
|---|
|[4]|
|203|
|305|

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print arr = set_difference(dynamic([1,2,3]), dynamic([1,2,3]))
```

|ARR|
|---|
|[]|