---
title: 'series_periods_validate (): Explorador de datos de Azure'
description: En este artículo se describe series_periods_validate () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2019
ms.openlocfilehash: b157c7fef0b9b4d98f08f5e5020803eea3960097
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372498"
---
# <a name="series_periods_validate"></a>series_periods_validate()

Comprueba si una serie temporal contiene patrones periódicos de longitud determinada.  

Muy a menudo, una métrica que mide el tráfico de una aplicación se caracteriza por períodos semanales o diarios. Esto se puede confirmar mediante la ejecución `series_periods_validate()` de la comprobación de períodos semanales y diarios.

La función toma como entrada una columna que contiene una matriz dinámica de series temporales (normalmente, la salida resultante del operador [Make-series](make-seriesoperator.md) ) y uno o más `real` números que definen las longitudes de los períodos que se van a validar. 

La función genera 2 columnas:
* *periods*: una matriz dinámica que contiene los períodos que se van a validar (suministrados en la entrada)
* *puntuaciones*: una matriz dinámica que contiene una puntuación entre 0 y 1 que mide la importancia de un punto en su posición respectiva en la matriz de *puntos* .

**Sintaxis**

`series_periods_validate(`*x* `,` *period1* [ `,` *Period2* `,` . . . ] `)`

**Argumentos**

* *x*: expresión escalar de matriz dinámica que es una matriz de valores numéricos, normalmente el resultado de los operadores [Make-series](make-seriesoperator.md) o [make_list](makelist-aggfunction.md) .
* *period1*, *Period2*, etc.: `real` números que especifican los períodos que se van a validar, en unidades del tamaño de la ubicación. Por ejemplo, si la serie está en las bandejas 1H, un período semanal es 168 bandejas.

> [!IMPORTANT]
> * El valor mínimo de cada uno de los argumentos *period* es **4** y el valor máximo es la mitad de la longitud de la serie de entrada. para un argumento *period* fuera de estos límites, la puntuación de salida será **0**.
>
> * La serie temporal de entrada debe ser normal, es decir, agregada en ubicaciones constantes (que siempre es el caso si se ha creado mediante [la creación de una serie](make-seriesoperator.md)). En caso contrario, el resultado no tendrá sentido.
> 
> * La función acepta hasta 16 períodos para la validación.


**Ejemplo**

La siguiente consulta inserta una instantánea de un mes del tráfico de una aplicación, agregada dos veces al día (es decir, el tamaño de la ubicación es de 12 horas).

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print y=dynamic([80,139,87,110,68,54,50,51,53,133,86,141,97,156,94,149,95,140,77,61,50,54,47,133,72,152,94,148,105,162,101,160,87,63,53,55,54,151,103,189,108,183,113,175,113,178,90,71,62,62,65,165,109,181,115,182,121,178,114,170])
| project x=range(1, array_length(y), 1), y  
| render linechart 
```

:::image type="content" source="images/series-periods/series-periods.png" alt-text="Períodos de la serie":::

`series_periods_validate()`La ejecución de esta serie para validar un período semanal (de 14 puntos) da como resultado una puntuación alta y una puntuación **0** al validar un período de cinco días (una longitud de 10 puntos).

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print y=dynamic([80,139,87,110,68,54,50,51,53,133,86,141,97,156,94,149,95,140,77,61,50,54,47,133,72,152,94,148,105,162,101,160,87,63,53,55,54,151,103,189,108,183,113,175,113,178,90,71,62,62,65,165,109,181,115,182,121,178,114,170])
| project x=range(1, array_length(y), 1), y  
| project series_periods_validate(y, 14.0, 10.0)
```

| períodos de la serie \_ \_ Validate \_ y \_ periods  | \_ \_ \_ puntuaciones de validación \_ y de períodos de serie |
|-------------|-------------------|
| [14,0, 10,0] | [0.84, 0.0]  |
