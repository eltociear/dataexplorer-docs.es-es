---
title: 'series_equals (): Explorador de datos de Azure'
description: En este artículo se describe series_equals () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: 85c6df85814d4b4aa4e1b00786d3d206d02116cd
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372825"
---
# <a name="series_equals"></a>series_equals()

Calcula la `==` operación de lógica de igualdad de elementos () de dos entradas de serie numéricas.

**Sintaxis**

`series_equals (`*Series1* `,` *Series2*`)`

**Argumentos**

* *Series1, series2*: matrices numéricas de entrada que se comparan en elementos. Todos los argumentos deben ser matrices dinámicas. 

**Devuelve**

Matriz dinámica de valores booleanos que contiene la operación lógica de igualdad de elementos calculados entre las dos entradas. Cualquier elemento no numérico o elemento no existente (matrices de distintos tamaños) produce un `null` valor de elemento.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print s1 = dynamic([1,2,4]), s2 = dynamic([4,2,1])
| extend s1_equals_s2 = series_equals(s1, s2)
```

|s1|s2|s1_equals_s2|
|---|---|---|
|[1, 2, 4]|[4, 2, 1]|[false, true, false]|

**Vea también**

Para ver las comparaciones de estadísticas completas de series, vea:
* [series_stats()](series-statsfunction.md)
* [series_stats_dynamic()](series-stats-dynamicfunction.md)
