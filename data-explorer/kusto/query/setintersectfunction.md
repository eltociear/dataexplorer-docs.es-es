---
title: set_intersect() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe set_intersect() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/02/2019
ms.openlocfilehash: 0a1ef86573a408f644e26b3b23f0db42e327573a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507759"
---
# <a name="set_intersect"></a>set_intersect()

Devuelve `dynamic` una matriz (JSON) del conjunto de todos los valores distintos que se encuentran en todas las matrices - (arr1 arr2 ....).

**Sintaxis**

`set_intersect(`*arr1*`, `*arr2*`[`,` *arr3*, ...])`

**Argumentos**

* *arr1... arrN*: Matrices de entrada para crear un conjunto de intersección (al menos dos matrices). Todos los argumentos deben ser matrices dinámicas (consulte [pack_array](packarrayfunction.md)). 

**Devuelve**

Devuelve una matriz dinámica del conjunto de todos los valores distintos que se encuentran en todas las matrices. Ver [`set_union()`](setunionfunction.md) [`set_difference()`](setdifferencefunction.md)y .

**Ejemplo**

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
|[2]|
|[3]|

```kusto
print arr = set_intersect(dynamic([1, 2, 3]), dynamic([4,5]))
```

|Arr|
|---|
|[]|