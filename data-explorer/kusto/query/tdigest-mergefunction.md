---
title: 'tdigest_merge (): Explorador de datos de Azure'
description: En este artículo se describe tdigest_merge () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/09/2019
ms.openlocfilehash: 92dce1a98cc0e24dcfbfcd7cb875fa370e3ae1d0
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737732"
---
# <a name="tdigest_merge"></a>tdigest_merge()

Combina `tdigest` los resultados (versión escalar de la versión [`tdigest_merge()`](tdigest-merge-aggfunction.md)de agregado).

Obtenga más información sobre el algoritmo subyacente (T-Digest) y el error Estimado [aquí](percentiles-aggfunction.md#estimation-error-in-percentiles).

**Sintaxis**

`merge_tdigests(`*Expr1* `,` *expr2*`, ...)`

`tdigest_merge(`*Expr1* `,` *Expr2* expr2`, ...)` : un alias.

**Argumentos**

* Columnas que tienen los `tdigest` valores que se van a combinar.

**Devuelve**

Resultado de la combinación de las `*Expr1*`columnas `*Expr2*`,,... `*ExprN*` a uno `tdigest`.

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