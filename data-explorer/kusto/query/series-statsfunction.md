---
title: series_stats() - Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe series_stats() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/10/2020
ms.openlocfilehash: 07aa5df7351a5d4be1522d39456423197bde508d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507929"
---
# <a name="series_stats"></a>series_stats()

`series_stats()`devuelve estadísticas para una serie en varias columnas.  

La `series_stats()` función toma una columna que contiene la matriz numérica dinámica como entrada y calcula las siguientes columnas:
* `min`: valor mínimo en la matriz de entrada
* `min_idx`: primera posición del valor mínimo en la matriz de entrada
* `max`: valor máximo en la matriz de entrada
* `max_idx`: primera posición del valor máximo en la matriz de entrada
* `avg`: valor medio de la matriz de entrada
* `variance`: varianza de la muestra de la matriz de entrada
* `stdev`: desviación estándar de la muestra de la matriz de entrada

> [!NOTE] 
> Esta función devuelve varias columnas por lo que no se puede usar como argumento para otra función.

**Sintaxis**

project `series_stats(` *x* `[,` *ignore_nonfinite* `])` o `series_stats(`extend *x* `)` Devuelve todas las columnas mencionadas anteriormente con los siguientes nombres: series_stats_x_min, series_stats_x_min_idx y etc.
 
proyecto`series_stats(`(m, mi)*x* `)` o extender (m, mi)`series_stats(`*x* `)` Devuelve las siguientes columnas: m (min) y mi (min_idx).

**Argumentos**

* *x*: Celda de matriz dinámica, que es una matriz de valores numéricos. 
* *ignore_nonfinite*: Indicador booleano `false`(opcional, predeterminado: ) que especifica si se deben calcular las estadísticas mientras se ignoran los valores no finitos (*null*, *NaN*, *inf*, etc.). Si se `false`establece en , `null` los valores devueltos serían si los valores no finitos están presentes en la matriz.

**Ejemplo**

```kusto
print x=dynamic([23,46,23,87,4,8,3,75,2,56,13,75,32,16,29]) 
| project series_stats(x)

```

|series_stats_x_min|series_stats_x_min_idx|series_stats_x_max|series_stats_x_max_idx|series_stats_x_avg|series_stats_x_stdev|series_stats_x_variance|
|---|---|---|---|---|---|---|
|2|8|87|3|32,8|28.5036338535483|812.457142857143|
