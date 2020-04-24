---
title: tdigest_merge() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe tdigest_merge() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/09/2019
ms.openlocfilehash: 988d7f05791723a823a5850f6865a780477f7bd4
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506382"
---
# <a name="tdigest_merge"></a>tdigest_merge()

Combina los resultados de tdigest (versión [`tdigest_merge()`](tdigest-merge-aggfunction.md)escalar de la versión agregada).

Lea más sobre el algoritmo subyacente (T-Digest) y el error estimado [aquí](percentiles-aggfunction.md#estimation-error-in-percentiles).

**Sintaxis**

`merge_tdigests(`*Expr1* `,` *Expr2*`, ...)`

`tdigest_merge(`*Expr1* `,` *Expr2* `, ...)` - Un alias.

**Argumentos**

* Columnas que tienen los tdigests que se van a combinar.

**Devuelve**

El resultado para fusionar `*Expr1*` `*Expr2*`las columnas , , ... `*ExprN*` a un tdigest.

**Ejemplos**

```kusto
range x from 1 to 10 step 1 
| extend y = x + 10
| summarize tdigestX = tdigest(x), tdigestY = tdigest(y)
| project merged = tdigest_merge(tdigestX, tdigestY)
| project percentile_tdigest(merged, 100, typeof(long))
```

|percentile_tdigest_merged|
|---|
|20|