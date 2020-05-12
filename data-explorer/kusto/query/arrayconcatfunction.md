---
title: 'array_concat (): Explorador de datos de Azure'
description: En este artículo se describe array_concat () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 681178cc12d145b1c574357e87ae4f7b33d736c4
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225656"
---
# <a name="array_concat"></a>array_concat()

Concatena un número de matrices dinámicas en una sola matriz.

**Sintaxis**

`array_concat(`*arr1* `[` , ` *arr2*, ...]` ) '

**Argumentos**

* *arr1... arrN*: matrices de entrada que se van a concatenar en una matriz dinámica. Todos los argumentos deben ser matrices dinámicas (vea [pack_array](packarrayfunction.md)). 

**Devuelve**

Matriz dinámica de matrices con arr1, arr2,..., arrN.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| extend a1 = pack_array(x,y,z), a2 = pack_array(x, y)
| project array_concat(a1, a2)
```

|Column1|
|---|
|[1, 2, 4, 1, 2]|
|[2, 4, 8, 2, 4]|
|[3, 6, 12, 3, 6]|
