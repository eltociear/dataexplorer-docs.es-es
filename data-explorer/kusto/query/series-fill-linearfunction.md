---
title: series_fill_linear() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_fill_linear() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 4f9a12d172a1580d6a9e575b78b14404dce7820e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81508660"
---
# <a name="series_fill_linear"></a>series_fill_linear()

Realiza la interpolación lineal de los valores que faltan en una serie.

Toma una expresión que contiene la matriz numérica dinámica como entrada, realiza la interpolación lineal para todas las instancias de missing_value_placeholder y devuelve la matriz resultante. Si el principio y el final de la matriz contienen missing_value_placeholder, se reemplazará con el valor más cercano que no sea missing_value_placeholder (se puede desactivar). Si toda la matriz consta de la missing_value_placeholder, la matriz se rellenará con constant_value o 0 si no se especifica.  

**Sintaxis**

`series_fill_linear(`*x* `[,` *missing_value_placeholder*` [,`*fill_edges**constant_value* fill_edges constant_value` [,``]]]))`
* Devolverá la interpolación lineal de serie de *x* utilizando parámetros especificados.
 

**Argumentos**

* *x*: expresión escalar de matriz dinámica que es una matriz de valores numéricos.
* *missing_value_placeholder*: parámetro opcional que especifica un marcador de posición para los "valores que faltan" que se reemplazarán. El valor `double`predeterminado es (*null*).
* *fill_edges*: valor booleano que indica si *missing_value_placeholder* al principio y al final de la matriz se debe reemplazar con el valor más cercano. *True de* forma predeterminada. Si se establece en *false,* se conservará *n.o missing_value_placeholder* al principio y al final de la matriz.
* *constant_value*: el parámetro opcional relevante sólo para matrices consta enteramente de valores *nulos,* que especifica el valor constante para rellenar la serie. El valor predeterminado es *0*. Establecer este parámetro `double`en (*null*) dejará efectivamente los valores *nulos* donde están.

**Notas**

* Para aplicar cualquier función de interpolación después de [make-series](make-seriesoperator.md) se recomienda especificar *null* como valor predeterminado: 

```kusto
make-series num=count() default=long(null) on TimeStamp in range(ago(1d), ago(1h), 1h) by Os, Browser
```

* El *missing_value_placeholder* puede ser de cualquier tipo que se convertirá en tipos de elemento reales. Por `double`lo tanto, ( `int`*null*), `long`(*null*) o (*null*) tienen el mismo significado.
* Si *missing_value_placeholder* missing_value_placeholder `double`es (*null*) (o simplemente se omite que tienen el mismo significado), entonces un resultado puede contener valores *nulos.* Utilice otras funciones de interpolación para rellenarlas. Actualmente, solo [series_outliers()](series-outliersfunction.md) admite valores *nulos* en matrices de entrada.
* La función conserva el tipo original de elementos de matriz. Si *x* contiene sólo elementos *int* o *long,* la interpolación lineal devolverá valores interpolados redondeados en lugar de los exactos.

**Ejemplo**

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

|Arr|without_args|with_edges|wo_edges|with_const|
|---|---|---|---|---|
|[null,111.0,null,36.0,41.0,null,null,16.0,61.0,33.0,null,null]|[111.0,111.0,73.5,36.0,41.0,32.667,24.333,16.0,61.0,33.0,33.0,33.0]|[111.0,111.0,73.5,36.0,41.0,32.667,24.333,16.0,61.0,33.0,33.0,33.0]|[null,111.0,73.5,36.0,41.0,32.667,24.333,16.0,61.0,33.0,null,null]|[111.0,111.0,73.5,36.0,41.0,32.667,24.333,16.0,61.0,33.0,33.0,33.0]|
|[null,111,null,36,41,null,null,16,61,33,null,null]|[111,111,73,36,41,32,24,16,61,33,33,33]|[111,111,73,36,41,32,24,16,61,33,33,33]|[null,111,73,36,41,32,24,16,61,33,null,null]|[111,111,74,38, 41,32,24,16,61,33,33,33]|
|[null,null,null,null]|[0.0,0.0,0.0,0.0]|[0.0,0.0,0.0,0.0]|[0.0,0.0,0.0,0.0]|[3.14159,3.14159,3.14159,3.14159]|