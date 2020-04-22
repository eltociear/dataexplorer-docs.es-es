---
title: percentile(), percentiles() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe percentile(), percentiles() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: ecbb56305cfc43033ca172071f48b25768de6d2f
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663655"
---
# <a name="percentile-percentiles"></a>percentil(), percentiles()

Devuelve una estimación para el [percentil](#nearest-rank-percentile) de rango más cercano especificado de la población definida por *Expr*. La precisión depende de la densidad de población en la región del percentil. Esta función sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

* `percentiles()`es `percentile()`como , pero calcula un número de valores percentiles (que es más rápido que calcular cada percentil individualmente).
* `percentilesw()`es `percentilew()`como , pero calcula un número de valores de percentil ponderados (que es más rápido que calcular cada percentil individualmente).
* `percentilew()`y `percentilesw()` permite calcular percentiles ponderados. Los percentiles ponderados calculan los percentiles dados de una manera `Weight` "ponderada", tratando cada valor como si fuera repetida veces en la entrada.

**Sintaxis**

resumir `percentile(` *Expr* `,` *Percentile*`)`

resumir `percentiles(` *Expr* `,` *Percentile1* [`,` *Percentile2*]`)`

resumir `percentiles_array(` *Expr* `,` *Percentile1* [`,` *Percentile2*]`)`

resumir `percentiles_array(`la matriz *Expr* `,` *Dynamic*`)`

resumir `percentilew(` *Expr* `,` *WeightExpr* `,` *Percentile*`)`

resumir `percentilesw(` *Expr* `,` *WeightExpr* `,` *Percentile1* [`,` *Percentile2*]`)`

resumir `percentilesw_array(` *Expr* `,` *WeightExpr* `,` *Percentile1* [`,` *Percentile2*]`)`

resumir `percentilesw_array(` *expr* `,` *WeightExpr* `,` Dynamic *array*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación.
* *WeightExpr*: Expresión que se utilizará como peso de valores para el cálculo de la agregación.
* *Percentil* es una constante doble que especifica el percentil.
* *Matriz dinámica:* lista de percentiles en una matriz dinámica de números enteros o de punto flotante.

**Devuelve**

Devuelve una estimación para *Expr* de los percentiles especificados en el grupo. 

**Ejemplos**

El valor `Duration` de eso es mayor que el 95% del conjunto de muestras y menor que el 5% del conjunto de muestras:

```kusto
CallDetailRecords | summarize percentile(Duration, 95) by continent
```

Calcular simultáneamente 5, 50 (mediana) y 95:

```kusto
CallDetailRecords 
| summarize percentiles(Duration, 5, 50, 95) by continent
```

:::image type="content" source="images/aggregations/percentiles.png" alt-text="Percentiles":::

Los resultados muestran que en Europa, el 5% de las llamadas son más cortas que 11,55s, el 50% de las llamadas son más cortas que 3 minutos, 18,46 segundos y el 95% de las llamadas son más cortas que 40 minutos 48 segundos.

Calcular varias estadísticas:

```kusto
CallDetailRecords 
| summarize percentiles(Duration, 5, 50, 95), avg(Duration)
```

## <a name="weighted-percentiles"></a>Percentiles ponderados

Supongamos que mide el tiempo (Duración) que tarda una acción en completarse una y otra vez. En lugar de registrar cada valor de la medición, se condensan registrando cada valor de Duración, redondeado a 100 ms y cuántas veces apareció ese valor redondeado (BucketSize).

Puede utilizar `summarize percentilesw(Duration, BucketSize, ...)` para calcular los percentiles especificados de una manera "ponderada", tratando cada valor de Duration como si se repitiera BucketSize times en la entrada, sin tener que materializar esos registros.

Ejemplo: Un cliente tiene un conjunto de `{ 1, 1, 2, 2, 2, 5, 7, 7, 12, 12, 15, 15, 15, 18, 21, 22, 26, 35 }`valores de latencia en milisegundos: .

Para reducir el ancho de banda y el `{ 10, 20, 30, 40, 50, 100 }`almacenamiento, realice la agregación previa a los siguientes buckets: , y cuente el número de eventos en cada bucket, lo que proporciona la siguiente tabla de Kusto:

:::image type="content" source="images/aggregations/percentilesw-table.png" alt-text="Tabla Percentilesw":::

Que se puede leer como:
 * 8 eventos en el bucket de 10 ms (correspondiente al subconjunto) `{ 1, 1, 2, 2, 2, 5, 7, 7 }`
 * 6 eventos en el bucket de 20 ms (correspondiente al subconjunto) `{ 12, 12, 15, 15, 15, 18 }`
 * 3 eventos en el bucket de 30 ms (correspondiente al subconjunto) `{ 21, 22, 26 }`
 * 1 evento en el bucket de 40 ms (correspondiente al subconjunto) `{ 35 }`

En este punto, los datos originales ya no están disponibles, y todo lo que tenemos es el número de eventos en cada bucket. Para calcular percentiles a partir `percentilesw()` de estos datos, utilice la función. Por ejemplo, para los percentiles 50, 75 y 99,9, utilice la siguiente consulta: 

```kusto
datatable (ReqCount:long, LatencyBucket:long) 
[ 
    8, 10, 
    6, 20, 
    3, 30, 
    1, 40 
]
| summarize percentilesw(LatencyBucket, ReqCount, 50, 75, 99.9) 
```

El resultado de la consulta anterior es:

:::image type="content" source="images/aggregations/percentilesw-result.PNG" alt-text="Resultado de Percentilesw":::

Tenga en cuenta que la `percentiles(LatencyBucket, 50, 75, 99.9)` consulta anterior corresponde a la función si los datos se gastaron en el siguiente formulario:

:::image type="content" source="images/aggregations/percentilesw-rawtable.png" alt-text="Tabla cruda Percentilesw":::

## <a name="getting-multiple-percentiles-in-an-array"></a>Obtención de varios percentiles en una matriz

Se pueden obtener varios percentiles como una matriz en una sola columna dinámica en lugar de varias columnas: 

```kusto
CallDetailRecords 
| summarize percentiles_array(Duration, 5, 25, 50, 75, 95), avg(Duration)
```

:::image type="content" source="images/aggregations/percentiles-array-result.png" alt-text="Resultados de la matriz Percentiles":::

Del mismo modo, los percentiles ponderados se pueden devolver como una matriz dinámica`percentilesw_array`

Percentiles `percentiles_array` para `percentilesw_array` y se pueden especificar en una matriz dinámica de números enteros o de punto flotante. La matriz debe ser constante, pero no tiene que ser literal.

```kusto
CallDetailRecords 
| summarize percentiles_array(Duration, dynamic([5, 25, 50, 75, 95])), avg(Duration)
```

```kusto
CallDetailRecords 
| summarize percentiles_array(Duration, range(0, 100, 5)), avg(Duration)
```

## <a name="nearest-rank-percentile"></a>Percentil de rango más cercano
*P*-ésimo percentil (0 < *P* <a 100) de una lista de valores ordenados (ordenados de menor a mayor) es el valor más pequeño de la lista de tal manera que el porcentaje *P* de los datos es menor o igual a ese valor[(del artículo de Wikipedia sobre percentiles)](https://en.wikipedia.org/wiki/Percentile#The_Nearest_Rank_method)

Defina *percentiles 0*-th para que sea el miembro más pequeño de la población.

>[!NOTE]
> Dada la naturaleza aproximada del cálculo, el valor devuelto real puede no ser miembro de la población.
> La definición de rango más cercano significa que *P*.50 no se ajusta a la [definición interpolativa de la mediana.](https://en.wikipedia.org/wiki/Median) Al evaluar la importancia de esta discrepancia para la aplicación específica, debe tenerse en cuenta el tamaño de la población y un error de [estimación.](#estimation-error-in-percentiles)

## <a name="estimation-error-in-percentiles"></a>Error de estimación en percentiles

El agregado de percentiles proporciona un valor aproximado mediante [T-Digest](https://github.com/tdunning/t-digest/blob/master/docs/t-digest-paper/histo.pdf). 

>[!NOTE]
> * Los límites en el error de estimación varían con el valor del percentil solicitado. La mejor precisión está en los extremos de la escala [0..100]. Los percentiles 0 y 100 son los valores mínimos y máximos exactos de la distribución. La precisión se reduce gradualmente hacia el centro de la escala. Es peor en la mediana y está limitado al 1%. 
> * Los límites de los errores se observan en el rango, no en el valor. Supongamos que percentile(X, 50) devolvió un valor de Xm. La estimación garantiza que al menos el 49% y como máximo el 51% de los valores de X sean menores o iguales a Xm. No hay límite teórico en la diferencia entre Xm y el valor mediano real de X.
> * La estimación a veces puede dar lugar a un valor preciso, pero no hay condiciones confiables para definir cuándo será el caso.
