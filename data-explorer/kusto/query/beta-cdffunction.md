---
title: beta_cdf() - Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe beta_cdf() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4faaeddc647d047755108f3c993db855a56de363
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517925"
---
# <a name="beta_cdf"></a>beta_cdf()

Devuelve la función de distribución beta acumulativa estándar.

```kusto
beta_cdf(0.2, 10.0, 50.0)
```

Si *la probabilidad* = `beta_cdf(`*x*,... `)`, `beta_inv(`entonces *la probabilidad*,... `)` *x*x  = .

La distribución beta se suele usar para estudiar la variación en el porcentaje de algo en varias muestras, como la fracción del día que las personas pasan viendo la televisión.

**Sintaxis**

`beta_cdf(`*x*`, `*beta* *alfa*`, ``)`

**Argumentos**

* *x*: Un valor en el que evaluar la función.
* *alfa*: Un parámetro de la distribución.
* *beta*: Un parámetro de la distribución.

**Devuelve**

* La función de [distribución beta acumulativa](https://en.wikipedia.org/wiki/Beta_distribution#Cumulative_distribution_function).

**Notas**

Si cualquier argumento no es numérico, beta_cdf() devuelve un valor nulo.

Si x < 0 o x > 1, beta_cdf() devuelve el valor NaN.

Si el valor alfa es 0 o beta a 0, beta_cdf() devuelve el valor NaN.

**Ejemplos**

```kusto
datatable(x:double, alpha:double, beta:double, comment:string)
[
    0.9, 10.0, 20.0, "Valid input",
    1.5, 10.0, 20.0, "x > 1, yields NaN",
    double(-10), 10.0, 20.0, "x < 0, yields NaN",
    0.1, double(-1.0), 20.0, "alpha is < 0, yields NaN"
]
| extend b = beta_cdf(x, alpha, beta)
```

|x|alpha|beta|comment|b|
|---|---|---|---|---|
|0.9|10|20|Entrada válida|0.999999999999959|
|1.5|10|20|x > 1, rinde NaN|NaN|
|-10|10|20|x < 0, produce NaN|NaN|
|0,1|-1|20|alfa es < 0, produce NaN|NaN|


**Vea también**


* Para calcular la inversa de la función de densidad de probabilidad acumulativa beta, véase [beta-inv()](./beta-invfunction.md).
* Para calcular la función de densidad de probabilidad, véase [beta-pdf()](./beta-pdffunction.md).