---
title: 'series_stats_dynamic (): Explorador de datos de Azure'
description: En este artículo se describe series_stats_dynamic () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/10/2020
ms.openlocfilehash: 87cee5244fb1276733d4cf44d0477cc3351b947c
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372452"
---
# <a name="series_stats_dynamic"></a>series_stats_dynamic()

Devuelve las estadísticas de una serie en un objeto dinámico.  

La `series_stats_dynamic()` función toma una columna que contiene una matriz numérica dinámica como entrada y genera un valor dinámico con el siguiente contenido:
* `min`: valor mínimo de la matriz de entrada.
* `min_idx`: primera posición del valor mínimo de la matriz de entrada
* `max`: valor máximo de la matriz de entrada.
* `max_idx`: primera posición del valor máximo de la matriz de entrada
* `avg`: valor promedio de la matriz de entrada.
* `variance`: varianza de muestra de la matriz de entrada
* `stdev`: desviación estándar de ejemplo de la matriz de entrada.

**Sintaxis**

`series_stats_dynamic(`*x* `[,` *ignore_nonfinite* x`])`

**Argumentos**

* *x*: celda de matriz dinámica que es una matriz de valores numéricos. 
* *ignore_nonfinite*: marca booleana (opcional, predeterminada: `false` ) que especifica si se deben calcular las estadísticas mientras se omiten los valores no finitos (*null*, *Nan*, *INF*, etc.). Si se establece en `false` el resultado devuelto es `null` si los valores no finitos están presentes en la matriz.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print x=dynamic([23,46,23,87,4,8,3,75,2,56,13,75,32,16,29]) 
| project stats=series_stats_dynamic(x)
```

|stats
|---|
|{"min": 2,0, "min_idx": 8, "Max": 87,0, "max_idx": 3, "AVG": 32,8, "stdev": 28.503633853548269, "Variance": 812.45714285714291}
