---
title: 'series_periods_detect (): Explorador de datos de Azure'
description: En este artículo se describe series_periods_detect () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2019
ms.openlocfilehash: a3f2a325b63306f7fec6b11eb3e684d3918bc7d5
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372519"
---
# <a name="series_periods_detect"></a>series_periods_detect()

Busca los períodos más significativos que existen en una serie temporal.  

Muy a menudo, una métrica que mide el tráfico de una aplicación se caracteriza por dos períodos significativos: semanal y diaria. Dada una serie temporal de este tipo, `series_periods_detect()` se detectarán dos períodos dominantes.  
La función toma como entrada una columna que contiene una matriz dinámica de series temporales (normalmente, la salida resultante del operador [Make-series](make-seriesoperator.md) ), dos `real` números que definen el tamaño de período mínimo y máximo (es decir, el número de ubicaciones, por ejemplo, para la ubicación 1H, el tamaño de un período diario sería 24) que se va a buscar y un `long` número que define el número total de períodos La función genera 2 columnas:
* *periods*: una matriz dinámica que contiene los períodos que se han encontrado (en unidades del tamaño de la ubicación), ordenados por sus puntuaciones
* *puntuaciones*: una matriz dinámica que contiene valores entre 0 y 1, cada una de las cuales mide la importancia de un punto en su posición respectiva en la matriz de *puntos* .
 
**Sintaxis**

`series_periods_detect(`*x* `,` *min_period* `,` *max_period* `,` *num_periods*`)`

**Argumentos**

* *x*: expresión escalar de matriz dinámica que es una matriz de valores numéricos, normalmente el resultado de los operadores [Make-series](make-seriesoperator.md) o [make_list](makelist-aggfunction.md) .
* *min_period*: un `real` número que especifica el período mínimo que se va a buscar.
* *max_period*: un `real` número que especifica el período máximo que se va a buscar.
* *num_periods*: un `long` número que especifica el número máximo de períodos necesario. Será la longitud de las matrices dinámicas de salida.

> [!IMPORTANT]
> * El algoritmo puede detectar períodos que contengan al menos 4 puntos y a la mitad de la longitud de la serie. 
>
> * Debe establecer el *min_period* un poco más abajo y *max_period* un poco por encima de los períodos que espera encontrar en la serie temporal. Por ejemplo, si tiene una señal agregada cada hora, y busca períodos diarios > y semanales (que serían 24 & 168 respectivamente), puede establecer *min_period*= 0,8 \* 24, *max_period*= 1,2 \* 168, manteniendo un 20% de margen alrededor de estos períodos.
>
> * La serie temporal de entrada debe ser normal, es decir, agregada en ubicaciones constantes (que siempre es el caso si se ha creado mediante [la creación de una serie](make-seriesoperator.md)). En caso contrario, el resultado no tendrá sentido.


**Ejemplo**

La siguiente consulta inserta una instantánea de un mes del tráfico de una aplicación, agregada dos veces al día (es decir, el tamaño de la ubicación es de 12 horas).

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print y=dynamic([80,139,87,110,68,54,50,51,53,133,86,141,97,156,94,149,95,140,77,61,50,54,47,133,72,152,94,148,105,162,101,160,87,63,53,55,54,151,103,189,108,183,113,175,113,178,90,71,62,62,65,165,109,181,115,182,121,178,114,170])
| project x=range(1, array_length(y), 1), y  
| render linechart 
```

:::image type="content" source="images/series-periods/series-periods.png" alt-text="Períodos de la serie":::

`series_periods_detect()`La ejecución de esta serie da como resultado el período semanal (longitud de 14 puntos):

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print y=dynamic([80,139,87,110,68,54,50,51,53,133,86,141,97,156,94,149,95,140,77,61,50,54,47,133,72,152,94,148,105,162,101,160,87,63,53,55,54,151,103,189,108,183,113,175,113,178,90,71,62,62,65,165,109,181,115,182,121,178,114,170])
| project x=range(1, array_length(y), 1), y  
| project series_periods_detect(y, 0.0, 50.0, 2)
```

| períodos de la serie \_ \_ detectar \_ \_ períodos y  | los períodos de la serie \_ \_ detectan \_ \_ puntuaciones de períodos y \_ |
|-------------|-------------------|
| [14.0, 0.0] | [0,84, 0,0]  |


Tenga en cuenta que el período diario que también se puede mostrar en el gráfico no se encontró porque el muestreo es demasiado grueso (tamaño de la ubicación de 12h), por lo que un período diario de 2 depósitos está por encima del tamaño mínimo del período de 4 puntos requeridos por el algoritmo.
