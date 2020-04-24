---
title: series_divide() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_divide() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 8e8b806c325da9bfce5f79ce5a5c4e5cfadaa838
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81508847"
---
# <a name="series_divide"></a>series_divide()

Calcula la división de elementos de dos entradas de serie numérica.

**Sintaxis**

`series_divide(`*series1* `,` *series2*`)`

**Argumentos**

* *series1, series2*: Matrizs numéricas de entrada, la primera en ser elemento dividido por el segundo en un resultado de matriz dinámica. Todos los argumentos deben ser matrices dinámicas. 

**Devuelve**

Matriz dinámica de la operación de división de elementos calculada entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices `null` de diferentes tamaños) produce un valor de elemento.

Nota: la serie de resultados es de tipo double, incluso si las entradas son enteros. La división por cero sigue la doble división por cero (por ejemplo, 2/0 produce double(+inf)).

**Ejemplo**

```kusto
range x from 1 to 3 step 1
| extend y = x * 2
| extend z = y * 2
| project s1 = pack_array(x,y,z), s2 = pack_array(z, y, x)
| extend s1_divide_s2 = series_divide(s1, s2)
```

|s1         |s2|        s1_divide_s2|
|---|---|---|
|[1,2,4]    |[4,2,1]|   [0.25,1.0,4.0]|
|[2,4,8]    |[8,4,2]|   [0.25,1.0,4.0]|
|[3,6,12]   |[12,6,3]|  [0.25,1.0,4.0]|