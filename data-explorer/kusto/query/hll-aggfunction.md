---
title: 'HLL () (función de agregación): Azure Explorador de datos'
description: En este artículo se describe HLL () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/15/2020
ms.openlocfilehash: cbe1b0639a0379fe84bc9c100a629bbadd9c3a63
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226574"
---
# <a name="hll-aggregation-function"></a>HLL () (función de agregación)

Calcula los resultados intermedios de [`dcount`](dcount-aggfunction.md) en el grupo, solo en el contexto de la agregación dentro de [resumir](summarizeoperator.md).

Obtenga información sobre el [algoritmo subyacente (*H*Yper*l*og*l*OG) y la precisión de estimación](dcount-aggfunction.md#estimation-accuracy).

**Sintaxis**

`summarize hll(`*`Expr`* `[,` *`Accuracy`*`])`

**Argumentos**

* *`Expr`*: Expresión que se utilizará para el cálculo de agregaciones. 
* *`Accuracy`*, si se especifica, controla el equilibrio entre la velocidad y la precisión.

  |Valor de precisión |Precisión  |Velocidad  |Error  |
  |---------|---------|---------|---------|
  |`0` | lowest | más rápida | 1,6% |
  |`1` | default  | pensado | 0,8% |
  |`2` | high | lento | 0,4%  |
  |`3` | high | lento | 0,28% |
  |`4` | extra alta | más lentas | 0,2% |
    
**Devuelve**

Resultados intermedios del recuento distintivo de *`Expr`* en el grupo.
 
**Sugerencias**

1. Puede usar la función de agregación [`hll_merge`](hll-merge-aggfunction.md) para combinar más de un `hll` resultado intermedio (solo funciona en la `hll` salida).

1. Puede utilizar la función [`dcount_hll`](dcount-hllfunction.md) , que calculará `dcount` a partir de `hll`  /  `hll_merge` las funciones de agregación.

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| summarize hll(DamageProperty) by bin(StartTime,10m)

```

|StartTime|`hll_DamageProperty`|
|---|---|
|2007-09-18 20:00:00.0000000|[[1024, 14], [-5473486921211236216,-6230876016761372746, 3953448761157777955, 4246796580750024372], []]|
|2007-09-20 21:50:00.0000000|[[1024, 14], [4835649640695509390], []]|
|2007-09-29 08:00.0000000|[[1024, 14], [4246796580750024372], []]|
|2007-12-30 16:00:00.0000000|[[1024, 14], [4246796580750024372,-8936707700542868125], []]|
