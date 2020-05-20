---
title: 'dcount_hll (): Explorador de datos de Azure'
description: En este artículo se describe dcount_hll () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/15/2019
ms.openlocfilehash: 1b1b0c2313f32044a7988e0992c00786885ce2aa
ms.sourcegitcommit: 974d5f2bccabe504583e387904851275567832e7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/18/2020
ms.locfileid: "83550306"
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
