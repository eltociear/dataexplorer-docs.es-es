---
title: 'series_pearson_correlation (): Explorador de datos de Azure'
description: En este artículo se describe series_pearson_correlation () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/31/2019
ms.openlocfilehash: 9187c10ad62b4d925bf6211e64657fba5ae17b63
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372506"
---
# <a name="series_pearson_correlation"></a>series_pearson_correlation()

Calcula el coeficiente de correlación de Pearson de dos entradas de serie numérica.

Vea: [coeficiente de correlación de Pearson](https://en.wikipedia.org/wiki/Pearson_correlation_coefficient).

**Sintaxis**

`series_pearson_correlation(`*Series1* `,` *Series2*`)`

**Argumentos**

* *Series1, series2*: matrices numéricas de entrada para calcular el coeficiente de correlación. Todos los argumentos deben ser matrices dinámicas de la misma longitud. 

**Devuelve**

Coeficiente de correlación de Pearson calculado entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices de distintos tamaños) produce un `null` resultado.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range s1 from 1 to 5 step 1 | extend s2 = 2*s1 // Perfect correlation
| summarize s1 = make_list(s1), s2 = make_list(s2)
| extend correlation_coefficient = series_pearson_correlation(s1,s2)
```

|s1|s2|correlation_coefficient|
|---|---|---|
|[1, 2, 3, 4, 5]|[2, 4, 6, 8, 10]|1|
