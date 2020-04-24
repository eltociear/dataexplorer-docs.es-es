---
title: percentile_tdigest() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe percentile_tdigest() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/10/2019
ms.openlocfilehash: 655a0693b8e04b1230f6b9e8fe247bd2d77b7ac6
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511244"
---
# <a name="percentile_tdigest"></a>percentile_tdigest()

Calcula el resultado del percentil a partir de los resultados de tdigest (que fue generado por [tdigest()](tdigest-aggfunction.md) o [tdigest_merge()](tdigest-merge-aggfunction.md))

**Sintaxis**

`percentile_tdigest(`*Expr* `,` *Percentile1* [`,` *typeLiteral*]`)`

`percentiles_array_tdigest(`*Expr* `,` *Percentile1* [`,` *Percentile2*] ... [`,` *Percentilan*]`)`

`percentiles_array_tdigest(`*Matriz Expr* `,` *Dynamic*`)`

**Argumentos**

* *Expr*: Expresión generada por [tdigest](tdigest-aggfunction.md) o [tdigest_merge()](tdigest-merge-aggfunction.md).
* *Percentil* es una constante doble que especifica el percentil.
* *typeLiteral*: Un literal de tipo `typeof(long)`opcional (por ejemplo, ). Si se proporciona, el conjunto de resultados será de este tipo. 
* *Matriz dinámica:* lista de percentiles en una matriz dinámica de números enteros o de punto flotante

**Devuelve**

El valor percentiles/percentilesw de cada valor en *Expr*.

**Sugerencias**

1) La función debe recibir al menos un uno por ciento (y`,` tal vez más, ver la sintaxis anterior: *Percentile1* [ *Percentile2*] ... [`,` *PercentilesN*]) y el resultado será una matriz dinámica que incluye los resultados. (como) [`percentiles()`](percentiles-aggfunction.md)
  
2) si sólo se proporcionó el uno por ciento y el tipo también se proporcionó, entonces el resultado será una columna del mismo tipo proporcionada con los resultados de ese porcentaje (todos los tdigests deben ser de ese tipo en este caso).

3) si *Expr* incluye tdigests de diferentes tipos, no proporcione el tipo y el resultado será de tipo dinámico. (ver ejemplos a continuación).

**Ejemplos**

```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty) by State
| project percentile_tdigest(tdigestRes, 100, typeof(int))
```

|percentile_tdigest_tdigestRes|
|---|
|0|
|62000000|
|110000000|
|1200000|
|250000|


```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty) by State
| project percentiles_array_tdigest(tdigestRes, range(0, 100, 50), typeof(int))
```

|percentile_tdigest_tdigestRes|
|---|
|[0,0,0]|
|[0,0,62000000]|
|[0,0,110000000]|
|[0,0,1200000]|
|[0,0,250000]|


```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty) by State
| union (StormEvents | summarize tdigestRes = tdigest(EndTime) by State)
| project percentile_tdigest(tdigestRes, 100)
```

|percentile_tdigest_tdigestRes|
|---|
|[0]|
|[62000000]|
|["2007-12-20T11:30:00.0000000Z"]|
|["2007-12-31T23:59:00.0000000Z"]|