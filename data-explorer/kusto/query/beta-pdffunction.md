---
title: 'beta_pdf (): Explorador de datos de Azure'
description: En este artículo se describe beta_pdf () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 8ebd4cb0ab8a5bffec717f83892a3ea11b35f409
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227645"
---
# <a name="beta_pdf"></a>beta_pdf()

Devuelve la función de versión beta de densidad de probabilidad.

```kusto
beta_pdf(0.2, 10.0, 50.0)
```

La distribución beta se suele usar para estudiar la variación en el porcentaje de algo en varias muestras, como la fracción del día que las personas pasan viendo la televisión.

**Sintaxis**

`beta_pdf(`*x* `, ` *alfa* `, ` *versión beta*`)`

**Argumentos**

* *x*: un valor en el que se va a evaluar la función.
* *Alpha*: un parámetro de la distribución.
* *beta*: un parámetro de la distribución.

**Devuelve**

* Función de densidad de la [versión beta de probabilidad](https://en.wikipedia.org/wiki/Beta_distribution#Probability_density_function).

**Notas**

Si algún argumento no es numérico, beta_pdf () devuelve el valor null.

Si x ≤ 0 o 1 ≤ x, beta_pdf () devuelve el valor NaN.

Si alfa ≤ 0 o beta ≤ 0, beta_pdf () devuelve el valor NaN.

**Ejemplos**

<!-- csl: https://help.kusto.windows.net/Samples -->
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
|1.5|10|20|x > 1, produce NaN|NaN|
|-10|10|20|x < 0, genera NaN|NaN|
|0,1|-1|20|alfa es < 0, genera NaN|NaN|

**Referencias**.

* Para calcular el inverso de la función de densidad de probabilidad acumulativa beta, consulte [beta-inv ()](./beta-invfunction.md).
* Para la función de distribución beta acumulativa estándar, consulte [beta-CDF ()](./beta-cdffunction.md).
