---
title: array_concat() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe array_concat() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: c66c17ab147eb3d6c5f749e7f28fad347a50ce22
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518758"
---
# <a name="array_concat"></a>array_concat()

Concatena un número de matrices dinámicas a una sola matriz.

**Sintaxis**

`array_concat(`*arr1*`[`` *arr2*, ...]`, )»

**Argumentos**

* *arr1... arrN*: Matrizs de entrada que se concatenarán en una matriz dinámica. Todos los argumentos deben ser matrices dinámicas (consulte [pack_array](packarrayfunction.md)). 

**Devuelve**

Matriz dinámica de matrices con arr1, arr2, ... , arrN.

**Ejemplo**

```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| extend a1 = pack_array(x,y,z), a2 = pack_array(x, y)
| project array_concat(a1, a2)
```

|Column1|
|---|
|[1,2,4,1,2]|
|[2,4,8,2,4]|
|[3,6,12,3,6]|