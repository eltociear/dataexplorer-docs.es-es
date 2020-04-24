---
title: series_stats_dynamic() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_stats_dynamic() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/10/2020
ms.openlocfilehash: b667af6d037b0b316bf18406e1fb49528e390213
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507912"
---
# <a name="series_stats_dynamic"></a>series_stats_dynamic()

Devuelve estadísticas de una serie en un objeto dinámico.  

La `series_stats_dynamic()` función toma una columna que contiene la matriz numérica dinámica como entrada y genera un valor dinámico con el siguiente contenido:
* `min`: valor mínimo en la matriz de entrada
* `min_idx`: primera posición del valor mínimo en la matriz de entrada
* `max`: valor máximo en la matriz de entrada
* `max_idx`: primera posición del valor máximo en la matriz de entrada
* `avg`: valor medio de la matriz de entrada
* `variance`: varianza de la muestra de la matriz de entrada
* `stdev`: desviación estándar de la muestra de la matriz de entrada

**Sintaxis**

`series_stats_dynamic(`*x* `[,` *ignore_nonfinite*`])`

**Argumentos**

* *x*: Celda de matriz dinámica que es una matriz de valores numéricos. 
* *ignore_nonfinite*: Indicador booleano `false`(opcional, predeterminado: ) que especifica si se deben calcular las estadísticas mientras se ignoran los valores no finitos (*null*, *NaN*, *inf*, etc.). Si se `false` establece en `null` el resultado devuelto es si los valores no finitos están presentes en la matriz.

**Ejemplo**

```kusto
print x=dynamic([23,46,23,87,4,8,3,75,2,56,13,75,32,16,29]) 
| project stats=series_stats_dynamic(x)
```

|stats
|---|
|"min": 2.0, "min_idx": 8, "max": 87.0, "max_idx": 3, "avg": 32.8, "stdev": 28.503633853548269, "variance": 812.45714285714291 á