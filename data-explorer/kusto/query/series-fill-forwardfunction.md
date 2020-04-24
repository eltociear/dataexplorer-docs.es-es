---
title: series_fill_forward() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_fill_forward() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 6c79733aa1abf001f52eb07606c866904e370906
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81508677"
---
# <a name="series_fill_forward"></a>series_fill_forward()

Realiza la interpolación de relleno hacia delante de los valores que faltan en una serie.

Toma una expresión que contiene una matriz numérica dinámica como entrada, reemplaza todas las instancias de missing_value_placeholder por el valor más cercano de su lado izquierdo distinto de missing_value_placeholder y devuelve la matriz resultante. Se conservan las instancias más a la izquierda de missing_value_placeholder.

**Sintaxis**

`series_fill_forward(`*x*`[, `*missing_value_placeholder*`])`
* Devolverá la serie *x* con todas las instancias de *missing_value_placeholder* rellenan hacia delante.

**Argumentos**

* *x*: expresión escalar de matriz dinámica que es una matriz de valores numéricos. 
* *missing_value_placeholder*: parámetro opcional que especifica un marcador de posición para los valores que faltan que se reemplazarán. El valor `double`predeterminado es (*null*).

**Notas**

* Para aplicar cualquier función de interpolación después de [make-series](make-seriesoperator.md) se recomienda especificar *null* como valor predeterminado: 

```kusto
make-series num=count() default=long(null) on TimeStamp in range(ago(1d), ago(1h), 1h) by Os, Browser
```

* El *missing_value_placeholder* puede ser de cualquier tipo que se convertirá en tipos de elemento reales. Por `double`lo tanto, ( `int`*null*), `long`(*null*) o (*null*) tienen el mismo significado.
* Si *missing_value_placeholder* missing_value_placeholder `double`es (*null*) (o simplemente se omite que tienen el mismo significado), entonces un resultado puede contener valores *nulos.* Utilice otras funciones de interpolación para rellenarlas. Actualmente, solo [series_outliers()](series-outliersfunction.md) admite valores *nulos* en matrices de entrada.
* Las funciones conservan el tipo original de elementos de matriz.

**Ejemplo**

```kusto
let data = datatable(arr: dynamic)
[
    dynamic([null,null,36,41,null,null,16,61,33,null,null])   
];
data 
| project arr, 
          fill_forward = series_fill_forward(arr)  

```

|Arr|fill_forward|
|---|---|
|[null,null,36,41,null,null,16,61,33,null,null]|[null,null,36,41,41,41,16,61,33,33,33]|
   
Se puede utilizar [series_fill_backward](series-fill-backwardfunction.md) o [series-fill-const](series-fill-constfunction.md) para completar la interpolación de la matriz anterior.