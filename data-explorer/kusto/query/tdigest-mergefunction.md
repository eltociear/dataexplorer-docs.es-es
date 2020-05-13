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
ms.openlocfilehash: 1281e2afdf9770975c6f6f74399f9815adaec045
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83371060"
---
# <a name="tdigest_merge"></a>tdigest_merge()

Combina `tdigest` los resultados (versión escalar de la versión de agregado [`tdigest_merge()`](tdigest-merge-aggfunction.md) ).

Obtenga más información sobre el algoritmo subyacente (T-Digest) y el error Estimado [aquí](percentiles-aggfunction.md#estimation-error-in-percentiles).

**Sintaxis**

`merge_tdigests(`*Expr1* `,` *Expr2*`, ...)`

`tdigest_merge(`*Expr1* `,` *Expr2* `, ...)` -Un alias.

**Argumentos**

* Columnas que tienen los `tdigest` valores que se van a combinar.

**Devuelve**

El resultado para combinar las columnas `*Expr1*` , `*Expr2*` ,... `*ExprN*` con una `tdigest` .

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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
