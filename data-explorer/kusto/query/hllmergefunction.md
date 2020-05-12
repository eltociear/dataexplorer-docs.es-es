---
title: 'hll_merge (): Explorador de datos de Azure'
description: En este artículo se describe hll_merge () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/15/2019
ms.openlocfilehash: 2cddd645b0b89b3356adabae26874f93b41f8815
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226557"
---
# <a name="hll_merge"></a>hll_merge()

Combina `hll` los resultados (versión escalar de la versión de agregado [`hll_merge()`](hll-merge-aggfunction.md) ).

Obtenga información sobre el [algoritmo subyacente (*H*Yper*l*og*l*OG) y la precisión de estimación](dcount-aggfunction.md#estimation-accuracy).

**Sintaxis**

`hll_merge(`*Expr1* `,` *Expr2*`, ...)`

**Argumentos**

* Columnas que tienen `hll` valores que se van a combinar.

**Devuelve**

Resultado para combinar las columnas `*Exrp1*` , `*Expr2*` ,... `*ExprN*` con un `hll` valor.

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/KustoMonitoringPersistentDatabase -->
```kusto
range x from 1 to 10 step 1 
| extend y = x + 10
| summarize hll_x = hll(x), hll_y = hll(y)
| project merged = hll_merge(hll_x, hll_y)
| project dcount_hll(merged)
```

|`dcount_hll_merged`|
|---|
|20|
