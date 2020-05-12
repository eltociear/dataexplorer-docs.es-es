---
title: 'percentile_tdigest (): Explorador de datos de Azure'
description: En este artículo se describe percentile_tdigest () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/10/2019
ms.openlocfilehash: f2e5e4aca145e0d78baddd7b1e34ab3ce6d047d1
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225010"
---
# <a name="percentile_tdigest"></a>percentile_tdigest()

Calcula el resultado del percentil a partir de los `tdigest` resultados (generados por [tdigest ()](tdigest-aggfunction.md) o [tdigest_merge ()](tdigest-merge-aggfunction.md)).

**Sintaxis**

`percentile_tdigest(`*`Expr`*`,`*Percentile1* [ `,` *typeLiteral*]`)`

`percentiles_array_tdigest(`*`Expr`*`,`*Percentile1* [ `,` *Percentile2*]... [ `,` *Percentile*]`)`

`percentiles_array_tdigest(`*`Expr`*`,`*Matriz dinámica*`)`

**Argumentos**

* *Expr*: expresión generada por [`tdigest`](tdigest-aggfunction.md) o [tdigest_merge ()](tdigest-merge-aggfunction.md).
* *Percentil* es una constante doble que especifica el percentil.
* *typeLiteral*: un literal de tipo opcional (por ejemplo, `typeof(long)` ). Si se proporciona, el conjunto de resultados será de este tipo. 
* *Matriz dinámica*: lista de percentiles de una matriz dinámica de números enteros o de punto flotante.

**Devuelve**

El valor percentiles/percentilesw de cada valor de *`Expr`* .

**Sugerencias**

* La función debe recibir al menos un porcentaje (y quizás más, vea la sintaxis anterior: *Percentile1* [ `,` *Percentile2*]... [ `,` *Percentile*]) y el resultado será una matriz dinámica que incluye los resultados. (como [`percentiles()`](percentiles-aggfunction.md) )
  
* Si solo se proporcionó un porcentaje y también se proporciona el tipo, el resultado será una columna del mismo tipo proporcionado con los resultados de ese porcentaje. En este caso, todas `tdigest` las funciones deben ser de ese tipo.

* Si *`Expr`* incluye `tdigest` funciones de tipos diferentes, no proporcione el tipo. El resultado será de tipo Dynamic. Vea los siguientes ejemplos.

**Ejemplos**

```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty) by State
| project percentile_tdigest(tdigestRes, 100, typeof(int))
```

|percentile_tdigest_tdigestRes|
|---|
|0|
|62 millones|
|110 millones|
|1,2 millones|
|250000|


```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty) by State
| project percentiles_array_tdigest(tdigestRes, range(0, 100, 50), typeof(int))
```

|percentile_tdigest_tdigestRes|
|---|
|[0, 0,0]|
|[0, 0, 62000000]|
|[0, 0, 110000000]|
|[0, 0, 1200000]|
|[0, 0, 250000]|


```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty) by State
| union (StormEvents | summarize tdigestRes = tdigest(EndTime) by State)
| project percentile_tdigest(tdigestRes, 100)
```

|percentile_tdigest_tdigestRes|
|---|
|[0]|
|[62 millones]|
|["2007-12-20T11:30:00.0000000 Z"]|
|["2007-12-31T23:59:00.0000000 Z"]|
