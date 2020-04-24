---
title: series_subtract() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe series_subtract() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 1e984bc35192da5d61448211c49ff582f225eb19
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507946"
---
# <a name="series_subtract"></a>series_subtract()

Calcula la resta de elementos de dos entradas de serie numérica.

**Sintaxis**

`series_subtract(`*series1* `,` *series2*`)`

**Argumentos**

* *series1, series2*: Matrizs numéricas de entrada, la segunda que se resta a los elementos de la primera en un resultado de matriz dinámica. Todos los argumentos deben ser matrices dinámicas. 

**Devuelve**

Matriz dinámica de la operación de resta calculada en cuanto a elementos entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices `null` de diferentes tamaños) produce un valor de elemento.

**Ejemplo**

```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| project s1 = pack_array(x,y,z), s2 = pack_array(z, y, x)
| extend s1_subtract_s2 = series_subtract(s1, s2)
```

|s1|s2|s1_subtract_s2|
|---|---|---|
|[1,2,4]|[4,2,1]|[-3,0,3]|
|[2,4,8]|[8,4,2]|[-6,0,6]|
|[3,6,12]|[12,6,3]|[-9,0,9]|