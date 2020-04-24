---
title: beta_inv() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe beta_inv() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 20bdf8bfc01ef3ac6c6a12f6a43d87fd7b5c07e6
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517891"
---
# <a name="beta_inv"></a>beta_inv()

Devuelve la inversa de la función de densidad beta de probabilidad beta acumulada.

```kusto
beta_inv(0.1, 10.0, 50.0)
```

Si *la probabilidad* = `beta_cdf(`*x*,... `)`, `beta_inv(`entonces *la probabilidad*,... `)` *x*x  = . 

La distribución beta se puede usar en la planeación de proyectos para modelar las horas de finalización probables según una hora de finalización y una variabilidad esperadas.

**Sintaxis**

`beta_inv(`*probabilidad**alfa*`, `*beta* `, ``)`

**Argumentos**

* *probabilidad*: Una probabilidad asociada con la distribución beta.
* *alfa*: Un parámetro de la distribución.
* *beta*: Un parámetro de la distribución.

**Devuelve**

* La inversa de la función de densidad de probabilidad acumulativa beta [beta_cdf()](./beta-cdffunction.md)

**Notas**

Si cualquier argumento no es numérico, beta_inv() devuelve un valor nulo.

Si el valor alfa es 0 o beta a 0, beta_inv() devuelve el valor nulo.

Si la probabilidad es 0 o la probabilidad > 1, beta_inv() devuelve el valor NaN.

Dado un valor para la probabilidad, beta_inv() busca ese valor x tal que beta_cdf(x, alfa, beta) - probabilidad.

**Ejemplos**

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
|0,1|-1|20|alfa es < 0, produce NaN|NaN|

**Vea también**

* Para calcular la función de distribución beta acumulativa, véase [beta-cdf()](./beta-cdffunction.md).
* Para calcular la función de densidad beta de probabilidad, véase [beta-pdf()](./beta-pdffunction.md).