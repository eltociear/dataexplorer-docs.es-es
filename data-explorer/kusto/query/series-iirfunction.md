---
title: 'series_iir (): Explorador de datos de Azure'
description: En este artículo se describe series_iir () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/20/2019
ms.openlocfilehash: 8b03970aacafef932f6397e64afdf871dc086bc1
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372622"
---
# <a name="series_iir"></a>series_iir()

Aplica un filtro de respuesta infinita al impulso en una serie.  

Toma una expresión que contiene una matriz numérica dinámica como entrada y aplica un filtro de [respuesta infinita al impulso](https://en.wikipedia.org/wiki/Infinite_impulse_response) . Al especificar los coeficientes de filtro, se puede usar, por ejemplo, para calcular la suma acumulativa de la serie, aplicar operaciones de suavizado, así como varios filtros de paso [alto](https://en.wikipedia.org/wiki/High-pass_filter), [de paso de banda](https://en.wikipedia.org/wiki/Band-pass_filter) y [de paso bajo](https://en.wikipedia.org/wiki/Low-pass_filter) . La función toma como entrada la columna que contiene la matriz dinámica y dos matrices dinámicas estáticas de los coeficientes *a* y *b* del filtro, y aplica el filtro en la columna. De este modo, genera una columna de matriz dinámica que contiene el resultado filtrado.  
 

**Sintaxis**

`series_iir(`*x* `,` *b* `,` *a*`)`

**Argumentos**

* *x*: celda de matriz dinámica que es una matriz de valores numéricos, normalmente la salida resultante de operadores de [creación de serie](make-seriesoperator.md) o de [make_list](makelist-aggfunction.md) .
* *b*: expresión constante que contiene los coeficientes del numerador del filtro (almacenado como una matriz dinámica de valores numéricos).
* *r*: expresión constante, como *b*. que contiene los coeficientes denominadores del filtro.

> [!IMPORTANT]
> El primer elemento de `a` (es decir, `a[0]` ) no debe debe ser cero (para evitar la división por 0; vea la fórmula que aparece a continuación).

**Más información sobre la fórmula recursiva del filtro**

* Dada una matriz de entrada X y coeficientes matrices a, b de longitudes n_a y n_b respectivamente, la función de transferencia del filtro, que genera la matriz de salida Y, se define mediante (vea también en Wikipedia):

<div align="center">
Y<sub>i</sub> = a<sub>0</sub><sup>-1</sup>(b<sub>0</sub>x<sub>i</sub> 
 + b<sub>1</sub>x<sub>i-1</sub> + ... + b<sub>n<sub>b</sub>-1</sub>X<sub>i-n<sub>b</sub>-1</sub> 
 - a<sub>1</sub>s-<sub>1</sub>-a<sub>2</sub>y<sub>i-2</sub> - ...-a<sub>-1<sub>a</sub></sub>y<sub>i-n<sub>a</sub>-1</sub>)
</div>

**Ejemplo**

El cálculo de la suma acumulativa puede realizarse mediante un filtro IIR con coeficientes *a*= [1,-1] y *b*= [1]:  

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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

Aquí se muestra cómo encapsularla en una función:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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
