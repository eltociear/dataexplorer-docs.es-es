---
title: 'series_fill_forward (): Explorador de datos de Azure'
description: En este artículo se describe series_fill_forward () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: dc421c8321985d001bb08ba85965cf017b1d51c6
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372785"
---
# <a name="series_fill_forward"></a>series_fill_forward()

Realiza una interpolación de relleno hacia delante de los valores que faltan en una serie.

Una expresión que contiene una matriz numérica dinámica es la entrada. La función reemplaza todas las instancias de missing_value_placeholder por el valor más cercano de su lado izquierdo que no sea missing_value_placeholder y devuelve la matriz resultante. Se conservan las instancias más a la izquierda de missing_value_placeholder.

**Sintaxis**

`series_fill_forward(`*x* `[, ` *missing_value_placeholder*`])`
* Devolverá la serie *x* con todas las instancias de *missing_value_placeholder* reenvíos rellenados.

**Argumentos**

* *x*: expresión escalar de matriz dinámica, que es una matriz de valores numéricos. 
* *missing_value_placeholder*: parámetro opcional, que especifica un marcador de posición para que se reemplace un valor que falta. El valor predeterminado es `double` (*null*).

**Notas**

* Especifique *null* como valor predeterminado para aplicar las funciones de interpolación después [de la serie make](make-seriesoperator.md): 

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
make-series num=count() default=long(null) on TimeStamp in range(ago(1d), ago(1h), 1h) by Os, Browser
```

* El *missing_value_placeholder* puede ser de cualquier tipo que se convertirá en tipos de elementos reales. Ambos `double` (*null*) `long` (*null)* y `int` (*null*) tienen el mismo significado.
* Si missing_value_placeholder es (NULL) (o se omite, lo que tiene el mismo significado), un resultado puede contener valores *null* . Para rellenar estos valores *null* , utilice otras funciones de interpolación. Actualmente solo [series_outliers ()](series-outliersfunction.md) admiten valores *null* en las matrices de entrada.
* Las funciones conservan el tipo original de los elementos de matriz.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
let data = datatable(arr: dynamic)
[
    dynamic([null,null,36,41,null,null,16,61,33,null,null])   
];
data 
| project arr, 
          fill_forward = series_fill_forward(arr)  

```

|`arr`|`fill_forward`|
|---|---|
|[null, null, 36, 41, null, null, 16, 61, 33, null, null]|[null, null, 36, 41, 41, 41, 16, 61, 33, 33, 33]|
   
Use [series_fill_backward](series-fill-backwardfunction.md) o [series-Fill-const](series-fill-constfunction.md) para completar la interpolación de la matriz anterior.
