---
title: series_periods_detect() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_periods_detect() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2019
ms.openlocfilehash: 0bc78f93270809774504c1eccbc9fa2bbce6c964
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663441"
---
# <a name="series_periods_detect"></a>series_periods_detect()

Encuentra los períodos más significativos que existen en una serie temporal.  

Muy a menudo una métrica que mide el tráfico de una aplicación se caracteriza por dos períodos significativos: un semanal y un diario. Dada esta serie `series_periods_detect()` temporal, detectará estos 2 períodos dominantes.  
La función toma como entrada una columna que contiene una matriz dinámica de `real` series temporales (normalmente la salida resultante del operador [make-series),](make-seriesoperator.md) dos números que definen el tamaño mínimo y máximo del `long` período (es decir, el número de ubicaciones, por ejemplo, para 1h bin el tamaño de un período diario sería 24) para buscar, y un número que define el número total de períodos para la función a buscar. La función genera 2 columnas:
* *períodos*: una matriz dinámica que contiene los períodos que se han encontrado (en unidades del tamaño de la ubicación), ordenados por sus puntuaciones
* *puntuaciones*: una matriz dinámica que contiene valores entre 0 y 1, cada uno mide la importancia de un período en su posición respectiva en la matriz *de puntos*
 
**Sintaxis**

`series_periods_detect(`*x* `,` *num_periods* *max_period* `,` *min_period* `,``)`

**Argumentos**

* *x*: Expresión escalar de matriz dinámica que es una matriz de valores numéricos, normalmente la salida resultante de los operadores [make-series](make-seriesoperator.md) o [make_list.](makelist-aggfunction.md)
* *min_period*: `real` Un número que especifica el período mínimo para buscar.
* *max_period*: `real` Un número que especifica el período máximo a buscar.
* *num_periods*: `long` Un número que especifica el número máximo requerido de períodos. Esta será la longitud de las matrices dinámicas de salida.

> [!IMPORTANT]
> * El algoritmo puede detectar períodos que contienen al menos 4 puntos y como máximo la mitad de la longitud de la serie. 
>
> * Usted debe establecer el *min_period* un poco más abajo y *max_period* un poco por encima de los períodos que espera encontrar en la serie temporal. Por ejemplo, si tiene una señal agregada por hora y busca períodos diarios de > y semanales (es decir, 24\*& 168 respectivamente), puede establecer *min_period*0,8 24, *max_period*1,2\*168 euros, dejando un 20% de margen alrededor de estos períodos.
>
> * La serie temporal de entrada debe ser regular, es decir, agregada en ubicaciones constantes (que siempre es el caso si se ha creado utilizando [make-series](make-seriesoperator.md)). En caso contrario, el resultado no tendrá sentido.


**Ejemplo**

La siguiente consulta incrusta una instantánea de un mes del tráfico de una aplicación, agregada dos veces al día (es decir, el tamaño de la ubicación es de 12 horas).

```kusto
print y=dynamic([80,139,87,110,68,54,50,51,53,133,86,141,97,156,94,149,95,140,77,61,50,54,47,133,72,152,94,148,105,162,101,160,87,63,53,55,54,151,103,189,108,183,113,175,113,178,90,71,62,62,65,165,109,181,115,182,121,178,114,170])
| project x=range(1, array_length(y), 1), y  
| render linechart 
```

:::image type="content" source="images/samples/series-periods.png" alt-text="Períodos de la serie":::

La `series_periods_detect()` ejecución de esta serie da como resultado el período semanal (14 puntos de largo):

```kusto
print y=dynamic([80,139,87,110,68,54,50,51,53,133,86,141,97,156,94,149,95,140,77,61,50,54,47,133,72,152,94,148,105,162,101,160,87,63,53,55,54,151,103,189,108,183,113,175,113,178,90,71,62,62,65,165,109,181,115,182,121,178,114,170])
| project x=range(1, array_length(y), 1), y  
| project series_periods_detect(y, 0.0, 50.0, 2)
```

| períodos\_de\_\_serie\_detectan y períodos  | períodos\_de\_\_la\_serie\_detectan puntuaciones de períodos y |
|-------------|-------------------|
| [14.0, 0.0] | [0.84, 0.0]  |


Tenga en cuenta que el período diario que también se puede ver en el gráfico no se encontró ya que el muestreo es demasiado grueso (tamaño de la bandeja de 12 horas) por lo que un período diario de 2 bins es debajo del tamaño mínimo del período de 4 puntos requerido por el algoritmo.