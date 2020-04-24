---
title: series_iir() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe series_iir() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/20/2019
ms.openlocfilehash: 96452265e07a8a8b057dc70bec520be6ab298dd3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81508303"
---
# <a name="series_iir"></a>series_iir()

Aplica un filtro De respuesta de impulso infinito en una serie.  

Toma una expresión que contiene una matriz numérica dinámica como entrada y aplica un filtro [Infinite Impulse Response.](https://en.wikipedia.org/wiki/Infinite_impulse_response) Al especificar los coeficientes de filtro, se puede utilizar, por ejemplo, para calcular la suma acumulada de la serie, para aplicar operaciones de suavizado, así como varios filtros de [paso alto,](https://en.wikipedia.org/wiki/High-pass_filter) [paso de banda](https://en.wikipedia.org/wiki/Band-pass_filter) y paso [bajo.](https://en.wikipedia.org/wiki/Low-pass_filter) La función toma como entrada la columna que contiene la matriz dinámica y dos matrices dinámicas estáticas de los coeficientes *a* y *b* del filtro, y aplica el filtro en la columna. De este modo, genera una columna de matriz dinámica que contiene el resultado filtrado.  
 

**Sintaxis**

`series_iir(`*x* `,` *b* `,` *a*`)`

**Argumentos**

* *x*: Celda de matriz dinámica que es una matriz de valores numéricos, normalmente la salida resultante de los operadores [make-series](make-seriesoperator.md) o [make_list.](makelist-aggfunction.md)
* *b*: Una expresión constante que contiene los coeficientes del numerador del filtro (almacenado como una matriz dinámica de valores numéricos).
* *a*: Una expresión constante, como *b*. que contiene los coeficientes denominadores del filtro.

> [!IMPORTANT]
> El primer `a` elemento de (es `a[0]`decir) no debe ser cero (para evitar la división por 0; véase la fórmula siguiente).

**Más información sobre la fórmula recursiva del filtro**

* Dada una matriz de entrada X y matrices de coeficientes a, b de longitudes n_a y n_b respectivamente, la función de transferencia del filtro, generando la matriz de salida Y, se define por (véase también en Wikipedia):

<div align="center">
Y<sub>i</sub> a<sub>0</sub><sup>-1</sup>(b<sub>0</sub>X<sub>i</sub> 
 + b<sub>1</sub>X<sub>i-1</sub> + ... + b<sub>n<sub>b</sub>-1</sub>X<sub>i-n<sub>b</sub>-1</sub> 
 - a<sub>1</sub>Y<sub>i-1</sub>-a<sub>2</sub>Y<sub>i-2</sub> - ... -<sub>a n<sub>a</sub>-1</sub>Y<sub>i-n a -1<sub>a</sub></sub>)
</div>

**Ejemplo**

El cálculo de la suma acumulada se puede realizar mediante el filtro iir con los coeficientes *a*[1,-1] y *b*a [1]:  

```kusto
let x = range(1.0, 10, 1);
print x=x, y = series_iir(x, dynamic([1]), dynamic([1,-1]))
| mv-expand x, y
```

| x | y |
|:--|:--|
|1.0|1.0|
|2.0|3.0|
|3.0|6.0|
|4.0|10.0|

A continuación se muestra cómo envolverlo en una función:

```kusto
let vector_sum=(x:dynamic)
{
  let y=array_length(x) - 1;
  toreal(series_iir(x, dynamic([1]), dynamic([1, -1]))[y])
};
print d=dynamic([0, 1, 2, 3, 4])
| extend dd=vector_sum(d)
```

|d            |dd  |
|-------------|----|
|`[0,1,2,3,4]`|`10`|