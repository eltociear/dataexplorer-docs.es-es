---
title: 'series_fill_const (): Explorador de datos de Azure'
description: En este artículo se describe series_fill_const () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 22e40c810242ee82701cf2d0e382a9f1910ed22d
ms.sourcegitcommit: 4f68d6dbfa6463dbb284de0aa17fc193d529ce3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82741727"
---
# <a name="series_fill_const"></a>series_fill_const()

Reemplaza los valores que faltan en una serie con un valor constante especificado.

Toma una expresión que contiene una matriz numérica dinámica como entrada, reemplaza todas las instancias de missing_value_placeholder por la constant_value especificada y devuelve la matriz resultante.

**Sintaxis**

`series_fill_const(`*x*`[, `*constant_value* `[,` *missing_value_placeholder* de constant_value x`]])`
* Devolverá la serie *x* con todas las instancias de *missing_value_placeholder* reemplazadas por *constant_value*.

**Argumentos**

* *x*: expresión escalar de matriz dinámica que es una matriz de valores numéricos.
* *constant_value*: parámetro que especifica un marcador de posición para que se reemplace un valor que falta. El valor predeterminado es *0*. 
* *missing_value_placeholder*: parámetro opcional que especifica un marcador de posición para que se reemplace un valor que falta. El valor predeterminado `double`es (*null*).

**Notas**
* Puede crear una serie que rellene con un valor `default = ` constante mediante la sintaxis *DefaultValue* (o simplemente omitir que asumirá 0). Para obtener más información, consulte [crear serie](make-seriesoperator.md).

```kusto
make-series num=count() default=-1 on TimeStamp in range(ago(1d), ago(1h), 1h) by Os, Browser
```
  
* Para aplicar cualquier función de interpolación después [de make-series](make-seriesoperator.md), especifique *null* como valor predeterminado: 

```kusto
make-series num=count() default=long(null) on TimeStamp in range(ago(1d), ago(1h), 1h) by Os, Browser
```
  
* El *missing_value_placeholder* puede ser de cualquier tipo, que se convertirá en tipos de elementos reales. Como `double`tal, (NULL *),* `long`(*null*) o `int`(*null*) tienen el mismo significado.
* La función conserva el tipo original de los elementos de la matriz. 

**Ejemplo**

```kusto
let data = datatable(`arr`: dynamic)
[
    dynamic([111,null,36,41,23,null,16,61,33,null,null])   
];
data 
| project arr, 
          fill_const1 = series_fill_const(arr, 0.0),
          fill_const2 = series_fill_const(arr, -1)  
```

|`arr`|`fill_const1`|`fill_const2`|
|---|---|---|
|[111, null, 36, 41, 23, null, 16, 61, 33, null, null]|[111, 0.0, 36, 41, 23, 0.0, 16, 61, 33, 0.0, 0.0]|[111,-1, 36, 41, 23,-1, 16, 61, 33,-1,-1]|