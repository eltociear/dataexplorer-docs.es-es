---
title: dcount_hll ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe dcount_hll () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/15/2019
ms.openlocfilehash: d4a76a30526f5fecbafafd735cf72de92aae7644
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225197"
---
# <a name="dcount_hll"></a>dcount_hll()

Calcula el DCont a partir de los resultados de HLL (generados por [HLL](hll-aggfunction.md) o [hll_merge](hll-merge-aggfunction.md)).

Obtenga información sobre el [algoritmo subyacente (*H*Yper*l*og*l*OG) y la precisión de estimación](dcount-aggfunction.md#estimation-accuracy).

**Sintaxis**

`dcount_hll(`*Argumento*`)`

**Argumentos**

* *Expr*: expresión generada por [HLL](hll-aggfunction.md) o [HLL-Merge](hll-merge-aggfunction.md)

**Devuelve**

El recuento distintivo de cada valor en *expr*

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| summarize hllRes = hll(DamageProperty) by bin(StartTime,10m)
| summarize hllMerged = hll_merge(hllRes)
| project dcount_hll(hllMerged)
```

|dcount_hll_hllMerged|
|---|
|315|