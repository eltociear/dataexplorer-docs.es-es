---
title: 'series_stats (): Explorador de datos de Azure'
description: En este artículo se describe series_stats () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/10/2020
ms.openlocfilehash: 3fe88a5d53faaca4512d614d3e62204ac26e6fc5
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372443"
---
# <a name="series_stats"></a>series_stats()

`series_stats()`Devuelve las estadísticas de una serie en varias columnas.  

La `series_stats()` función toma una columna que contiene una matriz numérica dinámica como entrada y calcula las columnas siguientes:
* `min`: valor mínimo de la matriz de entrada.
* `min_idx`: primera posición del valor mínimo de la matriz de entrada
* `max`: valor máximo de la matriz de entrada.
* `max_idx`: primera posición del valor máximo de la matriz de entrada
* `avg`: valor promedio de la matriz de entrada.
* `variance`: varianza de muestra de la matriz de entrada
* `stdev`: desviación estándar de ejemplo de la matriz de entrada.

> [!NOTE] 
> Esta función devuelve varias columnas, por lo que no se puede usar como argumento para otra función.

**Sintaxis**

el proyecto `series_stats(` *x* `[,` *ignore_nonfinite* `])` o Extend `series_stats(` *x* `)` devuelve todas las columnas mencionadas anteriormente con los nombres siguientes: series_stats_x_min, series_stats_x_min_idx, etc.
 
Project (m, mi) = `series_stats(` *x* `)` o Extend (m, mi) = `series_stats(` *x* `)` devuelve las columnas siguientes: m (min) y mi (min_idx).

**Argumentos**

* *x*: celda de matriz dinámica, que es una matriz de valores numéricos. 
* *ignore_nonfinite*: marca booleana (opcional, predeterminada: `false` ) que especifica si se deben calcular las estadísticas mientras se omiten los valores no finitos (*null*, *Nan*, *INF*, etc.). Si se establece en `false` , los valores devueltos serían `null` si los valores no finitos están presentes en la matriz.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print x=dynamic([23,46,23,87,4,8,3,75,2,56,13,75,32,16,29]) 
| project series_stats(x)

```

|series_stats_x_min|series_stats_x_min_idx|series_stats_x_max|series_stats_x_max_idx|series_stats_x_avg|series_stats_x_stdev|series_stats_x_variance|
|---|---|---|---|---|---|---|
|2|8|87|3|32,8|28.5036338535483|812.457142857143|
