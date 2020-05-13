---
title: 'series_less (): Explorador de datos de Azure'
description: En este artículo se describe series_less () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: b1f51f30825ceecfc025219f61d181c39ab0268f
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372599"
---
# <a name="series_less"></a>series_less()

Calcula la `<` operación de lógica menos () de elementos de dos entradas de serie numérica.

**Sintaxis**

`series_less (`*Series1* `,` *Series2*`)`

**Argumentos**

* *Series1, series2*: matrices numéricas de entrada que se comparan en elementos. Todos los argumentos deben ser matrices dinámicas. 

**Devuelve**

Matriz dinámica de valores booleanos que contiene la operación de menos lógica calculada de elemento calculada entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices de distintos tamaños) produce un `null` valor de elemento.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print s1 = dynamic([1,2,4]), s2 = dynamic([4,2,1])
| extend s1_less_s2 = series_less(s1, s2)
```

|s1|s2|s1_less_s2|
|---|---|---|
|[1, 2, 4]|[4, 2, 1]|[true, false, false]|

**Vea también**

Para ver las comparaciones de estadísticas completas de series, vea:
* [series_stats()](series-statsfunction.md)
* [series_stats_dynamic()](series-stats-dynamicfunction.md)
