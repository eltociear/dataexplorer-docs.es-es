---
title: series_periods_validate() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_periods_validate() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2019
ms.openlocfilehash: 8eba96e21513e776c984a356f88a705ca46485af
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663396"
---
# <a name="series_periods_validate"></a>series_periods_validate()

Comprueba si una serie temporal contiene patrones periódicos de longitudes determinadas.  

Muy a menudo una métrica que mide el tráfico de una aplicación se caracteriza por un período semanal y / o diario. Esto se puede confirmar `series_periods_validate()` mediante la comprobación de un período semanal y diario.

La función toma como entrada una columna que contiene una matriz dinámica de series temporales (normalmente la salida resultante del operador [make-series)](make-seriesoperator.md) y uno o varios `real` números que definen las longitudes de los períodos que se van a validar. 

La función genera 2 columnas:
* *períodos*: una matriz dinámica que contiene los períodos a validar (suministrado en la entrada)
* *puntuaciones*: una matriz dinámica que contiene una puntuación entre 0 y 1 que mide la importancia de un período en su posición respectiva en la matriz *de puntos*

**Sintaxis**

`series_periods_validate(`*x* `,` *período1* [ `,` *período2* `,` . . . ] `)`

**Argumentos**

* *x*: Expresión escalar de matriz dinámica que es una matriz de valores numéricos, normalmente la salida resultante de los operadores [make-series](make-seriesoperator.md) o [make_list.](makelist-aggfunction.md)
* *period1*, *period2*, `real` etc.: números que especifican los períodos a validar, en unidades del tamaño de ubicación. Por ejemplo, si la serie está en contenedores de 1h, un período semanal es de 168 ubicaciones.

> [!IMPORTANT]
> * El valor mínimo para cada uno de los argumentos de *período* es **4** y el máximo es la mitad de la longitud de la serie de entrada; para un argumento de *período* fuera de estos límites, la puntuación de salida será **0**.
>
> * La serie temporal de entrada debe ser regular, es decir, agregada en ubicaciones constantes (que siempre es el caso si se ha creado utilizando [make-series](make-seriesoperator.md)). En caso contrario, el resultado no tendrá sentido.
> 
> * La función acepta hasta 16 períodos para validar.


**Ejemplo**

La siguiente consulta incrusta una instantánea de un mes del tráfico de una aplicación, agregada dos veces al día (es decir, el tamaño de la ubicación es de 12 horas).

```kusto
print y=dynamic([80,139,87,110,68,54,50,51,53,133,86,141,97,156,94,149,95,140,77,61,50,54,47,133,72,152,94,148,105,162,101,160,87,63,53,55,54,151,103,189,108,183,113,175,113,178,90,71,62,62,65,165,109,181,115,182,121,178,114,170])
| project x=range(1, array_length(y), 1), y  
| render linechart 
```

:::image type="content" source="images/samples/series-periods.png" alt-text="Períodos de la serie":::

Ejecutar `series_periods_validate()` en esta serie para validar un período semanal (14 puntos de largo) resulta en una puntuación alta, y con una puntuación **0** al validar un período de cinco días (10 puntos de largo).

```kusto
print y=dynamic([80,139,87,110,68,54,50,51,53,133,86,141,97,156,94,149,95,140,77,61,50,54,47,133,72,152,94,148,105,162,101,160,87,63,53,55,54,151,103,189,108,183,113,175,113,178,90,71,62,62,65,165,109,181,115,182,121,178,114,170])
| project x=range(1, array_length(y), 1), y  
| project series_periods_validate(y, 14.0, 10.0)
```

| períodos\_de\_\_la serie\_validan períodos y  | períodos\_de\_\_la serie\_validan puntuaciones y |
|-------------|-------------------|
| [14.0, 10.0] | [0.84,0.0]  |