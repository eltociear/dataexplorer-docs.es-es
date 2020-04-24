---
title: series_fill_backward() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_fill_backward() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: b8c3bb09c7067112fd358c94fd46ca85bea31358
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81508745"
---
# <a name="series_fill_backward"></a>series_fill_backward()

Realiza la interpolación de relleno hacia atrás de los valores que faltan en una serie.

Toma una expresión que contiene una matriz numérica dinámica como entrada, reemplaza todas las instancias de missing_value_placeholder con el valor más cercano de su lado derecho distinto de missing_value_placeholder y devuelve la matriz resultante. Se conservan las instancias más a la derecha de missing_value_placeholder.

**Sintaxis**

`series_fill_backward(`*x*`[, `*missing_value_placeholder*`])`
* Devolverá la serie *x* con todas las instancias de *missing_value_placeholder* rellenan hacia atrás.

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
    dynamic([111,null,36,41,null,null,16,61,33,null,null])   
];
data 
| project arr, 
          fill_forward = series_fill_backward(arr)

```

|Arr|fill_forward|
|---|---|
|[111,null,36,41,null,null,16,61,33,null,null]|[111,36,36,41,16, 16,16,61,33,null,null]|

  
Se puede utilizar [series_fill_forward](series-fill-forwardfunction.md) o [series-fill-const](series-fill-constfunction.md) para completar la interpolación de la matriz anterior.