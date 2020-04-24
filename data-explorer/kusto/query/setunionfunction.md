---
title: set_union() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe set_union() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/02/2019
ms.openlocfilehash: de9220ea6f9e23e458dd379fb164317842a48c94
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507674"
---
# <a name="set_union"></a>set_union()

Devuelve `dynamic` una matriz (JSON) del conjunto de todos los valores distintos que se encuentran en cualquiera de las matrices - (arr1 arr2 ....).

**Sintaxis**

`set_union(`*arr1*`, `*arr2*`[`,` *arr3*, ...]``)`

**Argumentos**

* *arr1... arrN*: matrizs de entrada para crear un conjunto de uniones (al menos dos matrices). Todos los argumentos deben ser matrices dinámicas (consulte [pack_array](packarrayfunction.md)). 

**Devuelve**

Devuelve una matriz dinámica del conjunto de todos los valores distintos que se encuentran en cualquiera de las matrices. Ver [`set_intersect()`](setintersectfunction.md) [`set_difference()`](setdifferencefunction.md)y .

**Ejemplo**

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
|[1,2,4,8]|
|[2,4,8,16]|
|[3,6,12,24]|