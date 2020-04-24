---
title: series_pearson_correlation() - Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe series_pearson_correlation() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/31/2019
ms.openlocfilehash: 6454ec528e7a9e53b2feab5a7fefa1236ed80cdf
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81508116"
---
# <a name="series_pearson_correlation"></a>series_pearson_correlation()

Calcula el coeficiente de correlación pearson de dos entradas de serie numérica.

Véase: Coeficiente de [correlación de Pearson](https://en.wikipedia.org/wiki/Pearson_correlation_coefficient).

**Sintaxis**

`series_pearson_correlation(`*Series1* `,` *Series2*`)`

**Argumentos**

* *Series1, Series2*: Matrizs numéricas de entrada para calcular el coeficiente de correlación. Todos los argumentos deben ser matrices dinámicas de la misma longitud. 

**Devuelve**

El coeficiente de correlación de Pearson calculado entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices `null` de diferentes tamaños) produce un resultado.

**Ejemplo**

```kusto
range s1 from 1 to 5 step 1 | extend s2 = 2*s1 // Perfect correlation
| summarize s1 = make_list(s1), s2 = make_list(s2)
| extend correlation_coefficient = series_pearson_correlation(s1,s2)
```

|s1|s2|correlation_coefficient|
|---|---|---|
|[1,2,3,4,5]|[2,4,6,8,10]|1|
