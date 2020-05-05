---
title: 'series_fill_backward (): Explorador de datos de Azure'
description: En este artículo se describe series_fill_backward () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 211ac6cb078e2f61243f4616f9141a6f4834d464
ms.sourcegitcommit: 4f68d6dbfa6463dbb284de0aa17fc193d529ce3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82741800"
---
# <a name="series_fill_backward"></a>series_fill_backward()

Realiza una interpolación de relleno hacia atrás de los valores que faltan en una serie.

Una expresión que contiene una matriz numérica dinámica es la entrada. La función reemplaza todas las instancias de missing_value_placeholder por el valor más cercano del lado derecho (distinto de missing_value_placeholder) y devuelve la matriz resultante. Se conservan las instancias de missing_value_placeholder situadas más a la derecha.

**Sintaxis**

`series_fill_backward(`*x*`[, `*missing_value_placeholder* x`])`
* Devolverá la serie *x* con todas las instancias de *missing_value_placeholder* rellenadas hacia atrás.

**Argumentos**

* *x*: expresión escalar de matriz dinámica, que es una matriz de valores numéricos.
* *missing_value_placeholder*: este parámetro opcional especifica un marcador de posición para los valores que faltan. El valor predeterminado es `double`(*null*).

**Notas**

* Especifique *null* como valor predeterminado para aplicar las funciones de interpolación después [de make-series](make-seriesoperator.md): 

```kusto
make-series num=count() default=long(null) on TimeStamp in range(ago(1d), ago(1h), 1h) by Os, Browser
```

* El *missing_value_placeholder* puede ser de cualquier tipo que se convertirá en tipos de elementos reales. Ambos `double`(*null*), `long`(*null*) y `int`(*null*) tienen el mismo significado.
* Si *missing_value_placeholder* es `double`(*null*) (o se omite, que tiene el mismo significado), un resultado puede contener valores *null* . Para rellenar estos valores *null* , utilice otras funciones de interpolación. Actualmente solo [series_outliers ()](series-outliersfunction.md) admiten valores *null* en las matrices de entrada.
* La función conserva el tipo original de los elementos de matriz.

**Ejemplo**

```kusto
let data = datatable(arr: dynamic)
[
    dynamic([111,null,36,41,null,null,16,61,33,null,null])   
];
data 
| project arr, 
          fill_forward = series_fill_backward(arr)

```

|`arr`|`fill_forward`|
|---|---|
|[111, null, 36, 41, null, null, 16, 61, 33, null, null]|[111, 36, 36, 41, 16, 16, 16, 61, 33, null, null]|

  
Use [series_fill_forward](series-fill-forwardfunction.md) o [series-Fill-const](series-fill-constfunction.md) para completar la interpolación de la matriz anterior.
