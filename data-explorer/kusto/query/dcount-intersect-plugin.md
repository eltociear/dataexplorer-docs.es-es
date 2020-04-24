---
title: dcount_intersect complemento - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe dcount_intersect complemento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 7771456ffa75085c79933c2e789e3d98f352b76f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516140"
---
# <a name="dcount_intersect-plugin"></a>dcount_intersect plugin

Calcula la intersección entre N conjuntos en función de los valores hll (N en el rango de [2..16]) y devuelve N valores dcount.

Dados los sets S<sub>1</sub>, S<sub>2</sub>, .. S<sub>n</sub> - devuelve valores que representarán recuentos distintos de:  
S<sub>1</sub>, S<sub>1</sub> s<sub>2</sub>,  
S<sub>1</sub> s<sub>2</sub> s<sub>s 3</sub>,  
... ,  
S<sub>1</sub> s<sub>2</sub> ... S<sub>n</sub>

    T | evaluate dcount_intersect(hll_1, hll_2, hll_3)

**Sintaxis**

*T* `| evaluate` T `dcount_intersect(` *hll_1*,`,` *hll_2*, [ *hll_3* `,` ...]`)`

**Argumentos**

* *T*: La expresión tabular de entrada.
* *hll_i*: los valores del set S<sub>i</sub> calculados con la función [hll().](./hll-aggfunction.md)

**Devuelve**

Devuelve una tabla con N dcount valores (por columnas de columna, que representan establece intersecciones).
Los nombres de columna son s0, s1, ... (hasta n-1).

**Ejemplos**

```kusto
// Generate numbers from 1 to 100
range x from 1 to 100 step 1
| extend isEven = (x % 2 == 0), isMod3 = (x % 3 == 0), isMod5 = (x % 5 == 0)
// Calculate conditional HLL values (note that '0' is included in each of them as additional value, so we will subtract it later)
| summarize hll_even = hll(iif(isEven, x, 0), 2),
            hll_mod3 = hll(iif(isMod3, x, 0), 2),
            hll_mod5 = hll(iif(isMod5, x, 0), 2) 
// Invoke the plugin that calculates dcount intersections         
| evaluate dcount_intersect(hll_even, hll_mod3, hll_mod5)
| project evenNumbers = s0 - 1,             //                             100 / 2 = 50
          even_and_mod3 = s1 - 1,           // gcd(2,3) = 6, therefor:     100 / 6 = 16
          even_and_mod3_and_mod5 = s2 - 1   // gcd(2,3,5) is 30, therefore: 100 / 30 = 3 
```

|evenNumbers|even_and_mod3|even_and_mod3_and_mod5|
|---|---|---|
|50|16|3|