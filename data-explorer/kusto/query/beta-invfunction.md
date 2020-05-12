---
title: beta_inv ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe beta_inv () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 60b054bbd234a77f81c47e375b98be0a5df103a5
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227662"
---
# <a name="beta_inv"></a>beta_inv()

Devuelve el inverso de la función de densidad beta acumulativa de la versión beta.

```kusto
beta_inv(0.1, 10.0, 50.0)
```

Si la *probabilidad*  =  `beta_cdf(` *x*,... `)` , la `beta_inv(` *probabilidad*,... `)`  =  *x*. 

La distribución beta se puede usar en la planeación de proyectos para modelar las horas de finalización probables según una hora de finalización y una variabilidad esperadas.

**Sintaxis**

`beta_inv(`*probabilidad* `, ` *alfa* `, ` *versión beta*`)`

**Argumentos**

* *probabilidad*: una probabilidad asociada a la distribución beta.
* *Alpha*: un parámetro de la distribución.
* *beta*: un parámetro de la distribución.

**Devuelve**

* El inverso de la función de densidad de probabilidad acumulativa beta [beta_cdf ()](./beta-cdffunction.md)

**Notas**

Si algún argumento no es numérico, beta_inv () devuelve el valor null.

Si alfa ≤ 0 o beta ≤ 0, beta_inv () devuelve el valor null.

Si la probabilidad ≤ 0 o la probabilidad > 1, beta_inv () devuelve el valor NaN.

Dado un valor de probabilidad, beta_inv () busca ese valor x tal que beta_cdf (x, alfa, beta) = probabilidad.

**Ejemplos**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(p:double, alpha:double, beta:double, comment:string)
[
    0.1, 10.0, 20.0, "Valid input",
    1.5, 10.0, 20.0, "p > 1, yields null",
    0.1, double(-1.0), 20.0, "alpha is < 0, yields NaN"
]
| extend b = beta_inv(p, alpha, beta)
```

|p|alpha|beta|comment|b|
|---|---|---|---|---|
|0,1|10|20|Entrada válida|0.226415022388749|
|1.5|10|20|p > 1, produce null||
|0,1|-1|20|alfa es < 0, genera NaN|NaN|

**Vea también**

* Para calcular la función de distribución acumulativa de la versión beta, consulte [beta-CDF ()](./beta-cdffunction.md).
* Para la función de densidad de la versión beta de probabilidad, consulte [beta-pdf ()](./beta-pdffunction.md).
