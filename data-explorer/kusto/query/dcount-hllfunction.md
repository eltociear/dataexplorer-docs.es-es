---
title: dcount_hll() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe dcount_hll() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/15/2019
ms.openlocfilehash: a0c921efa90f5d66fe42fa6ee872204b5bb399cd
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516157"
---
# <a name="dcount_hll"></a>dcount_hll()

Calcula el dcount a partir de los resultados hll (que fue generado por [hll](hll-aggfunction.md) o [hll_merge](hll-merge-aggfunction.md)).

Lea sobre el [algoritmo subyacente (*H*yper*L*og*L*og) y](dcount-aggfunction.md#estimation-accuracy)la precisión de estimación .

**Sintaxis**

`dcount_hll(`*Expr*`)`

**Argumentos**

* *Expr*: Expresión generada por [hll](hll-aggfunction.md) o [hll-merge](hll-merge-aggfunction.md)

**Devuelve**

El recuento diferenciado de cada valor en *Expr*

**Ejemplos**

```kusto
StormEvents
| summarize hllRes = hll(DamageProperty) by bin(StartTime,10m)
| summarize hllMerged = hll_merge(hllRes)
| project dcount_hll(hllMerged)
```

|dcount_hll_hllMerged|
|---|
|315|