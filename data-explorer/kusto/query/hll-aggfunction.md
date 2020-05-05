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
ms.openlocfilehash: f5c47e2ebd2acc0b2ec250d183d65b6536aff756
ms.sourcegitcommit: 4f68d6dbfa6463dbb284de0aa17fc193d529ce3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82741825"
---
# <a name="hll-aggregation-function"></a>HLL () (función de agregación)

Calcula los resultados intermedios [`dcount`](dcount-aggfunction.md) de en el grupo, solo en el contexto de la agregación dentro de [resumir](summarizeoperator.md).

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

Resultados intermedios del recuento *`Expr`* distintivo de en el grupo.
 
**Sugerencias**

1. Puede usar la función [`hll_merge`](hll-merge-aggfunction.md) de agregación para combinar más de `hll` un resultado intermedio (solo funciona `hll` en la salida).

1. Puede [`dcount_hll`](dcount-hllfunction.md)utilizar la función, que calculará `dcount` a partir de `hll`  /  `hll_merge` las funciones de agregación.

**Ejemplos**

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