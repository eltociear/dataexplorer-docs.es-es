---
title: hll_merge() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe hll_merge() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/15/2019
ms.openlocfilehash: 10e726af4e9dd2e129b526f016a7c37dc0c99d50
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514100"
---
# <a name="hll_merge"></a>hll_merge()

Combina los resultados de hll (versión [`hll_merge()`](hll-merge-aggfunction.md)escalar de la versión agregada).

Lea sobre el [algoritmo subyacente (*H*yper*L*og*L*og) y](dcount-aggfunction.md#estimation-accuracy)la precisión de estimación .

**Sintaxis**

`hll_merge(`*Expr1* `,` *Expr2*`, ...)`

**Argumentos**

* Columnas que tienen los valores hll que se van a fusionar.

**Devuelve**

El resultado para fusionar `*Exrp1*` `*Expr2*`las columnas , , ... `*ExprN*` a un valor hll.

**Ejemplos**

```kusto
range x from 1 to 10 step 1 
| extend y = x + 10
| summarize hll_x = hll(x), hll_y = hll(y)
| project merged = hll_merge(hll_x, hll_y)
| project dcount_hll(merged)
```

|dcount_hll_merged|
|---|
|20|