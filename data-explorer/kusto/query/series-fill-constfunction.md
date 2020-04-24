---
title: series_fill_const() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_fill_const() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: c7f5f939068737bdd6519fc0c63b663d19ae076a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81508779"
---
# <a name="series_fill_const"></a>series_fill_const()

Reemplaza los valores que faltan en una serie por un valor constante especificado.

Toma una expresión que contiene la matriz numérica dinámica como entrada, reemplaza todas las instancias de missing_value_placeholder con constant_value especificadas y devuelve la matriz resultante.

**Sintaxis**

`series_fill_const(`*x*`[, ` *missing_value_placeholder* *constant_value* `[,``]])`
* Devolverá la serie *x* con todas las instancias de *missing_value_placeholder* reemplazadas por *constant_value*.

**Argumentos**

* *x*: expresión escalar de matriz dinámica que es una matriz de valores numéricos.
* *constant_value*: parámetro que especifica un marcador de posición para los valores que faltan que se van a reemplazar. El valor predeterminado es *0*. 
* *missing_value_placeholder*: parámetro opcional que especifica un marcador de posición para los valores que faltan que se reemplazarán. El valor `double`predeterminado es (*null*).

**Notas**
* Es posible crear una serie con relleno `default = ` constante en una llamada utilizando la sintaxis *DefaultValue* (o simplemente omitiendo que asumirá 0). Consulte [make-series](make-seriesoperator.md) para obtener más información.

```kusto
make-series num=count() default=-1 on TimeStamp in range(ago(1d), ago(1h), 1h) by Os, Browser
```
  
* Para aplicar cualquier función de interpolación después de [make-series](make-seriesoperator.md) se recomienda especificar *null* como valor predeterminado: 

```kusto
make-series num=count() default=long(null) on TimeStamp in range(ago(1d), ago(1h), 1h) by Os, Browser
```
  
* El *missing_value_placeholder* puede ser de cualquier tipo que se convertirá en tipos de elemento reales. Por `double`lo tanto, ( `int`*null*), `long`(*null*) o (*null*) tienen el mismo significado.
* La función conserva el tipo original de los elementos de matriz. 

**Ejemplo**

```kusto
let data = datatable(arr: dynamic)
[
    dynamic([111,null,36,41,23,null,16,61,33,null,null])   
];
data 
| project arr, 
          fill_const1 = series_fill_const(arr, 0.0),
          fill_const2 = series_fill_const(arr, -1)  
```

|Arr|fill_const1|fill_const2|
|---|---|---|
|[111,null,36,41,23,null,16,61,33,null,null]|[111,0.0,36,41,23,0.0,16,61,33,0.0,0.0]|[111,-1,36,41,23,-1,16,61,33,-1,-1]|