---
title: 'series_less_equals (): Explorador de datos de Azure'
description: En este artículo se describe series_less_equals () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: 8f86d7174e73f2ffbace935f695c1220572a0e38
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372606"
---
# <a name="series_less_equals"></a>series_less_equals()

Calcula la operación de lógica inferior o igual de elemento ( `<=` ) de dos entradas numéricas de serie.

**Sintaxis**

`series_less_equals (`*Series1* `,` *Series2*`)`

**Argumentos**

* *Series1, series2*: matrices numéricas de entrada que se comparan en elementos. Todos los argumentos deben ser matrices dinámicas. 

**Devuelve**

Matriz dinámica de valores booleanos que contiene la operación de lógica menos o igual de elemento calculada entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices de distintos tamaños) produce un `null` valor de elemento.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print s1 = dynamic([1,2,4]), s2 = dynamic([4,2,1])
| extend s1_less_equals_s2 = series_less_equals(s1, s2)
```

|s1|s2|s1_less_equals_s2|
|---|---|---|
|[1, 2, 4]|[4, 2, 1]|[true, true, false]|

**Vea también**

Para ver las comparaciones de estadísticas completas de series, vea:
* [series_stats()](series-statsfunction.md)
* [series_stats_dynamic()](series-stats-dynamicfunction.md)
