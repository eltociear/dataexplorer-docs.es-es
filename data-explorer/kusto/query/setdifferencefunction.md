---
title: set_difference() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe set_difference() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/02/2019
ms.openlocfilehash: d4edb8ec46fca99b7dd58b11bbd54442a9340c7a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507810"
---
# <a name="set_difference"></a>set_difference()

Devuelve `dynamic` una matriz (JSON) del conjunto de todos los valores distintos que se encuentran en la primera matriz pero que no están en otras matrices - (((arr1 árr2) á arr3) arr3) ....).

**Sintaxis**

`set_difference(`*arr1*`, `*arr2*`[`,` *arr3*, ...])`

**Argumentos**

* *arr1... arrN*: Matrizs de entrada para crear un conjunto de diferencias (al menos dos matrices). Todos los argumentos deben ser matrices dinámicas (consulte [pack_array](packarrayfunction.md)). 

**Devuelve**

Devuelve una matriz dinámica del conjunto de todos los valores distintos que están en arr1 pero no en otras matrices. Ver [`set_union()`](setunionfunction.md) [`set_intersect()`](setintersectfunction.md)y .

**Ejemplo**

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
|[8]|
|[12]|

```kusto
print arr = set_difference(dynamic([1,2,3]), dynamic([1,2,3]))
```

|Arr|
|---|
|[]|