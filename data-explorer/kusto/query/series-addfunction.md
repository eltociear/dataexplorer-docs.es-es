---
title: series_add() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_add() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: ea8c391060e487413a166c236abf789edcba0a0c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509068"
---
# <a name="series_add"></a>series_add()

Calcula la adición de elementos de dos entradas de serie numérica.

**Sintaxis**

`series_add(`*series1* `,` *series2*`)`

**Argumentos**

* *series1, series2*: Introduzca matrices numéricas que se agregarán a un resultado de matriz dinámica. Todos los argumentos deben ser matrices dinámicas. 

**Devuelve**

Matriz dinámica de la operación de adición de elementos calculados entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices `null` de diferentes tamaños) produce un valor de elemento.

**Ejemplo**

```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| project s1 = pack_array(x,y,z), s2 = pack_array(z, y, x)
| extend s1_add_s2 = series_add(s1, s2)
```

|s1|s2|s1_add_s2|
|---|---|---|
|[1,2,4]|[4,2,1]|[5,4,5]|
|[2,4,8]|[8,4,2]|[10,8,10]|
|[3,6,12]|[12,6,3]|[15,12,15]|