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
ms.openlocfilehash: 35dab83a658b714a220c7fbd6ff751627c0741e4
ms.sourcegitcommit: 4f68d6dbfa6463dbb284de0aa17fc193d529ce3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82741786"
---
# <a name="hll_merge"></a>hll_merge()

Combina `hll` los resultados (versión escalar de la versión [`hll_merge()`](hll-merge-aggfunction.md)de agregado).

Obtenga información sobre el [algoritmo subyacente (*H*Yper*l*og*l*OG) y la precisión de estimación](dcount-aggfunction.md#estimation-accuracy).

**Sintaxis**

`hll_merge(`*Expr1* `,` *expr2*`, ...)`

**Argumentos**

* Columnas que tienen `hll` valores que se van a combinar.

**Devuelve**

Resultado de la combinación de las `*Exrp1*`columnas `*Expr2*`,,... `*ExprN*` a un `hll` valor.

**Ejemplos**

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