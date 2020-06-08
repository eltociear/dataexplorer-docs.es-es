---
title: percentil (), percentiles ()-Azure Explorador de datos
description: En este artículo se describe percentil (), percentiles () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: dfb957189eb0d9be552cf12b32ef57452a375c51
ms.sourcegitcommit: 188f89553b9d0230a8e7152fa1fce56c09ebb6d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2020
ms.locfileid: "84512561"
---
# <a name="percentile-percentiles"></a>percentil (), percentiles ()

Devuelve una estimación para el [percentil de rango más cercano](#nearest-rank-percentile) especificado de la población definida por `*Expr*` .
La precisión depende de la densidad de población en la región del percentil. Esta función solo se puede usar en el contexto de la agregación dentro de [resumir](summarizeoperator.md)

* `percentiles()`es como `percentile()` , pero calcula un número de valores de percentil, que es más rápido que calcular cada percentil individualmente.
* `percentilesw()`es como `percentilew()` , pero calcula un número de valores de percentil ponderados, que es más rápido que calcular cada percentil individualmente.
* `percentilew()`y `percentilesw()` permiten calcular los percentiles ponderados. Los percentiles ponderados calculan los percentiles determinados de un modo "ponderado", al tratar cada valor como si estuvieran repetidos `weight` veces, en la entrada.

**Sintaxis**

resumir `percentile(` *Expr* `,` *percentil* de expresión`)`

resumir `percentiles(` *expr* `,` *Percentile1* [ `,` *Percentile2*]`)`

resumir `percentiles_array(` *expr* `,` *Percentile1* [ `,` *Percentile2*]`)`

resumir `percentiles_array(` *Expr* `,` *matriz dinámica* de expresión`)`

resumir `percentilew(` *expresión* `,` *WeightExpr* `,` *percentil*`)`

resumir `percentilesw(` *expr* `,` *WeightExpr* `,` *Percentile1* [ `,` *Percentile2*]`)`

resumir `percentilesw_array(` *expr* `,` *WeightExpr* `,` *Percentile1* [ `,` *Percentile2*]`)`

resumir `percentilesw_array(` *Expr* `,` *WeightExpr* `,` *matriz dinámica* de expr WeightExpr`)`

**Argumentos**

* `*Expr*`: Expresión que se utilizará para el cálculo de agregaciones.
* `*WeightExpr*`: Expresión que se usará como el peso de los valores para el cálculo de agregaciones.
* `*Percentile*`: Una constante Double que especifica el percentil.
* `*Dynamic array*`: lista de percentiles de una matriz dinámica de números enteros o de punto flotante.

**Devuelve**

Devuelve una estimación de `*Expr*` los percentiles especificados en el grupo. 

**Ejemplos**

El valor de `Duration` que es mayor que el 95% del conjunto de muestra y menor que el 5% del conjunto de muestra.

```kusto
CallDetailRecords | summarize percentile(Duration, 95) by continent
```

Calcular simultáneamente 5, 50 (mediana) y 95.

```kusto
CallDetailRecords 
| summarize percentiles(Duration, 5, 50, 95) by continent
```

:::image type="content" source="images/percentiles-aggfunction/percentiles.png" alt-text="Percentiles":::

Los resultados muestran que en Europa, el 5% de las llamadas son más cortos que 11.55 s, el 50% de las llamadas son más cortos que 3 minutos, 18,46 segundos y el 95% de las llamadas son más cortos que 40 minutos 48 segundos.

```kusto
CallDetailRecords 
| summarize percentiles(Duration, 5, 50, 95), avg(Duration)
```

## <a name="weighted-percentiles"></a>Percentiles ponderados

Supongamos que mide repetidamente el tiempo (duración) que tarda en completarse la acción. En lugar de grabar cada valor de la medida, registre cada valor de duración, redondeado a 100 ms y el número de veces que aparece el valor redondeado (BucketSize).

`summarize percentilesw(Duration, BucketSize, ...)`Se usa para calcular los percentiles determinados de un modo "ponderado". Trate cada valor de Duration como si estuviera repetido BucketSize veces en la entrada, sin necesidad de materializar realmente esos registros.

**Ejemplo**

Un cliente tiene un conjunto de valores de latencia en milisegundos: `{ 1, 1, 2, 2, 2, 5, 7, 7, 12, 12, 15, 15, 15, 18, 21, 22, 26, 35 }` .

Para reducir el ancho de banda y el almacenamiento, realice la agregación previa a los cubos siguientes: `{ 10, 20, 30, 40, 50, 100 }` . Cuente el número de eventos de cada depósito para generar la siguiente tabla:

:::image type="content" source="images/percentiles-aggfunction/percentilesw-table.png" alt-text="Tabla Percentilesw":::

En la tabla se muestra:
 * Ocho eventos en el depósito de 10 ms (que corresponde al subconjunto `{ 1, 1, 2, 2, 2, 5, 7, 7 }` )
 * Seis eventos en el depósito de 20 ms (que corresponde al subconjunto `{ 12, 12, 15, 15, 15, 18 }` )
 * Tres eventos en el depósito de 30 ms (que corresponde al subconjunto `{ 21, 22, 26 }` )
 * Un evento en el depósito 40-ms (correspondiente al subconjunto `{ 35 }` )

En este momento, los datos originales ya no están disponibles. Solo el número de eventos de cada depósito. Para calcular los percentiles a partir de estos datos, use la `percentilesw()` función.
Por ejemplo, para los percentiles 50, 75 y 99,9, utilice la siguiente consulta.

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

:::image type="content" source="images/percentiles-aggfunction/percentilesw-result.png" alt-text="Resultado de Percentilesw" border="false":::


La consulta anterior corresponde a la función `percentiles(LatencyBucket, 50, 75, 99.9)` , si los datos se han expandido de la forma siguiente:

:::image type="content" source="images/percentiles-aggfunction/percentilesw-rawtable.png" alt-text="Percentilesw tabla sin formato":::

## <a name="getting-multiple-percentiles-in-an-array"></a>Obtener varios percentiles en una matriz

Se pueden obtener varios percentiles como una matriz en una sola columna dinámica, en lugar de en varias columnas.

```kusto
CallDetailRecords 
| summarize percentiles_array(Duration, 5, 25, 50, 75, 95), avg(Duration)
```

:::image type="content" source="images/percentiles-aggfunction/percentiles-array-result.png" alt-text="Percentiles resultado de la matriz":::

Del mismo modo, los percentiles ponderados se pueden devolver como una matriz dinámica mediante `percentilesw_array` .

Los percentiles de `percentiles_array` y `percentilesw_array` se pueden especificar en una matriz dinámica de números enteros o de punto flotante. La matriz debe ser constante, pero no tiene que ser literal.

```kusto
CallDetailRecords 
| summarize percentiles_array(Duration, dynamic([5, 25, 50, 75, 95])), avg(Duration)
```

```kusto
CallDetailRecords 
| summarize percentiles_array(Duration, range(0, 100, 5)), avg(Duration)
```

## <a name="nearest-rank-percentile"></a>Percentil de rango más cercano

*P*-ésimo percentil (0 < *P* <= 100) de una lista de valores ordenados, ordenados de menor a mayor, es el valor más pequeño de la lista. El *p* por ciento de los datos es menor o igual que el valor *p*-ésimo percentil ([del artículo de la Wikipedia sobre percentiles](https://en.wikipedia.org/wiki/Percentile#The_Nearest_Rank_method)).

Defina *0*--percentiles como el miembro más pequeño de la población.

>[!NOTE]
> Dada la naturaleza aproximada del cálculo, el valor devuelto real no puede ser un miembro del rellenado.
> La definición del rango más cercano significa que *P*= 50 no se ajusta a la [definición interpolativa de la mediana](https://en.wikipedia.org/wiki/Median). Al evaluar la importancia de esta discrepancia para la aplicación específica, se debe tener en cuenta el tamaño de la población y un [error de estimación](#estimation-error-in-percentiles) .

## <a name="estimation-error-in-percentiles"></a>Error de estimación en percentiles

El agregado de percentiles proporciona un valor aproximado mediante [T-Digest](https://github.com/tdunning/t-digest/blob/master/docs/t-digest-paper/histo.pdf).

>[!NOTE]
> * Los límites en el error de estimación varían con el valor del percentil solicitado. La mejor precisión se encuentra en ambos extremos de la escala [0.. 100]. Percentiles 0 y 100 son los valores mínimo y máximo exactos de la distribución. La precisión se reduce gradualmente hacia el centro de la escala. Es peor en la mediana y se limita al 1%.
> * Los límites de los errores se observan en el rango, no en el valor. Suponga que percentil (X, 50) devolvió un valor de XM. La estimación garantiza que al menos el 49% y, como máximo, el 51% de los valores de X son menores o iguales a XM. No hay ningún límite teórico sobre la diferencia entre XM y el valor medio real de X.
> * A veces, la estimación puede dar lugar a un valor preciso, pero no hay condiciones confiables que definir cuando sea el caso.
