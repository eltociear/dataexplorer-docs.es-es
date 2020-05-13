---
title: 'series_fill_linear (): Explorador de datos de Azure'
description: En este artículo se describe series_fill_linear () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 4cec053990457a6b33c7446c5b32c63713320de9
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372759"
---
# <a name="series_fill_linear"></a>series_fill_linear()

Interpola linealmente los valores que faltan en una serie.

Toma una expresión que contiene una matriz numérica dinámica como entrada, realiza la interpolación lineal para todas las instancias de missing_value_placeholder y devuelve la matriz resultante. Si el principio y el final de la matriz contienen missing_value_placeholder, se reemplazará por el valor más cercano que no sea missing_value_placeholder. Esta característica se puede desactivar. Si toda la matriz está formada por el missing_value_placeholder, la matriz se rellenará con constant_value, o 0 si no se especifica.  

**Sintaxis**

`series_fill_linear(`*x* `[,` *missing_value_placeholder* ` [,` *fill_edges* ` [,` *constant_value*`]]]))`
* Devolverá la interpolación lineal de la serie *x* utilizando los parámetros especificados.
 

**Argumentos**

* *x*: expresión escalar de matriz dinámica, que es una matriz de valores numéricos.
* *missing_value_placeholder*: parámetro opcional, que especifica un marcador de posición para los "valores que faltan" que se van a reemplazar. El valor predeterminado es `double` (*null*).
* *fill_edges*: valor booleano, que indica si *missing_value_placeholder* al principio y al final de la matriz deben reemplazarse por el valor más cercano. *True* de forma predeterminada. Si se establece en *false*, se conservará *missing_value_placeholder* al principio y al final de la matriz.
* *constant_value*: el parámetro opcional solo es relevante para las matrices que constan por completo de valores *null* . Este parámetro especifica un valor constante con el que se va a rellenar la serie. El valor predeterminado es *0*. Establecer este parámetro en `double` (*null*) dejará los valores *null* de forma efectiva.

**Notas**

* Para aplicar las funciones de interpolación después [de make-series](make-seriesoperator.md), especifique *null* como valor predeterminado: 

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
make-series num=count() default=long(null) on TimeStamp in range(ago(1d), ago(1h), 1h) by Os, Browser
```

* El *missing_value_placeholder* puede ser de cualquier tipo que se convertirá en tipos de elementos reales. Como tal, `double` (NULL*null*), `long` (*null*) o `int` (*null*) tienen el mismo significado.
* Si *missing_value_placeholder* es `double` (*null*) (o se omite, que tiene el mismo significado), un resultado puede contener valores *null* . Utilice otras funciones de interpolación para rellenar estos valores *null* . Actualmente solo [series_outliers ()](series-outliersfunction.md) admiten valores *null* en las matrices de entrada.
* La función conserva el tipo original de los elementos de matriz. Si x contiene solo elementos int o Long, la interpolación lineal devolverá valores interpolados redondeados en lugar de los exactos.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
let data = datatable(arr: dynamic)
[
    dynamic([null, 111.0, null, 36.0, 41.0, null, null, 16.0, 61.0, 33.0, null, null]), // Array of double    
    dynamic([null, 111,   null, 36,   41,   null, null, 16,   61,   33,   null, null]), // Similar array of int
    dynamic([null, null, null, null])                                                   // Array with missing values only
];
data
| project arr, 
          without_args = series_fill_linear(arr),
          with_edges = series_fill_linear(arr, double(null), true),
          wo_edges = series_fill_linear(arr, double(null), false),
          with_const = series_fill_linear(arr, double(null), true, 3.14159)  

```

|`arr`|`without_args`|`with_edges`|`wo_edges`|`with_const`|
|---|---|---|---|---|
|[null, 111.0, null, 36.0, 41.0, null, null, 16,0, 61.0, 33.0, null, null]|[111.0, 111.0, 73.5, 36.0, 41.0, 32.667, 24.333, 16,0, 61.0, 33.0, 33.0, 33.0]|[111.0, 111.0, 73.5, 36.0, 41.0, 32.667, 24.333, 16,0, 61.0, 33.0, 33.0, 33.0]|[null, 111.0, 73.5, 36.0, 41.0, 32.667, 24.333, 16,0, 61.0, 33.0, null, null]|[111.0, 111.0, 73.5, 36.0, 41.0, 32.667, 24.333, 16,0, 61.0, 33.0, 33.0, 33.0]|
|[null, 111, null, 36, 41, null, null, 16, 61, 33, null, null]|[111111, 73, 36, 41, 32, 24, 16, 61, 33, 33, 33]|[111111, 73, 36, 41, 32, 24, 16, 61, 33, 33, 33]|[null, 111, 73, 36, 41, 32, 24, 16, 61, 33, null, null]|[111111, 74, 38, 41, 32, 24, 16, 61, 33, 33, 33]|
|[null, null, null, null]|[0.0, 0.0, 0.0, 0.0]|[0.0, 0.0, 0.0, 0.0]|[0.0, 0.0, 0.0, 0.0]|[3,14159, 3,14159, 3,14159, 3,14159]|
