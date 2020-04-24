---
title: beta_pdf() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe beta_pdf() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 53b86d88b05cc6c5cc31f1e3bbb9e3e712eed7f8
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517874"
---
# <a name="beta_pdf"></a>beta_pdf()

Devuelve la función beta de densidad de probabilidad.

```kusto
beta_pdf(0.2, 10.0, 50.0)
```

La distribución beta se suele usar para estudiar la variación en el porcentaje de algo en varias muestras, como la fracción del día que las personas pasan viendo la televisión.

**Sintaxis**

`beta_pdf(`*x*`, `*beta* *alfa*`, ``)`

**Argumentos**

* *x*: Un valor en el que evaluar la función.
* *alfa*: Un parámetro de la distribución.
* *beta*: Un parámetro de la distribución.

**Devuelve**

* La función de [densidad beta de probabilidad](https://en.wikipedia.org/wiki/Beta_distribution#Probability_density_function).

**Notas**

Si cualquier argumento no es numérico, beta_pdf() devuelve un valor nulo.

Si x es 0 o 1 x, beta_pdf() devuelve el valor NaN.

Si el valor alfa es 0 o beta a 0, beta_pdf() devuelve el valor NaN.

**Ejemplos**

```kusto
datatable(x:double, alpha:double, beta:double, comment:string)
[
    0.5, 10.0, 20.0, "Valid input",
    1.5, 10.0, 20.0, "x > 1, yields NaN",
    double(-10), 10.0, 20.0, "x < 0, yields NaN",
    0.1, double(-1.0), 20.0, "alpha is < 0, yields NaN"
]
| extend r = beta_pdf(x, alpha, beta)
```

|x|alpha|beta|comment|r|
|---|---|---|---|---|
|0.5|10|20|Entrada válida|0.746176019310951|
|1.5|10|20|x > 1, rinde NaN|NaN|
|-10|10|20|x < 0, produce NaN|NaN|
|0,1|-1|20|alfa es < 0, produce NaN|NaN|

**Referencias**

* Para calcular la inversa de la función de densidad de probabilidad acumulativa beta, véase [beta-inv()](./beta-invfunction.md).
* Para la función de distribución beta acumulativa estándar, véase [beta-cdf()](./beta-cdffunction.md).