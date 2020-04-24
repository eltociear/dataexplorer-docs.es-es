---
title: hll() (función de agregación) - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe hll() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/15/2020
ms.openlocfilehash: 52eac2984ed29bf8de21fb378a84b789015aa1c5
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514134"
---
# <a name="hll-aggregation-function"></a>hll() (función de agregación)

Calcula los resultados intermedios de [dcount](dcount-aggfunction.md) en todo el grupo. 

* Solo se puede utilizar en el contexto de la agregación dentro de [resumir](summarizeoperator.md).

Lea sobre el [algoritmo subyacente (*H*yper*L*og*L*og) y la precisión](dcount-aggfunction.md#estimation-accuracy)de estimación .

**Sintaxis**

`summarize``hll(` *Expr* `,` [ *Precisión*]`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. 
* *Accuracy*, si se especifica, controla el equilibrio entre velocidad y precisión.
    * `0` = el cálculo menos preciso y más rápido. Error del 1,6%
    * `1`• el valor por defecto, que equilibra la precisión y el tiempo de cálculo; alrededor de 0.8% error.
    * `2`• cálculo preciso y lento; alrededor de 0.4% error.
    * `3`• cálculo extra preciso y lento; alrededor de 0.28% de error.
    * `4`• cálculo súper preciso y más lento; alrededor de 0.2% de error.
    
**Devuelve**

Los resultados intermedios del recuento distinto de *Expr* en todo el grupo.
 
**Sugerencias**

1) Puede utilizar la función de agregación [hll_merge](hll-merge-aggfunction.md) para combinar más de un hll resultados intermedios (solo funciona en la salida hll).

2) Puede utilizar la función [dcount_hll](dcount-hllfunction.md) que calculará el dcount a partir de funciones de agregación hll / hll_merge.

**Ejemplos**

```kusto
StormEvents
| summarize hll(DamageProperty) by bin(StartTime,10m)

```

|StartTime|hll_DamageProperty|
|---|---|
|2007-09-18 20:00:00.0000000|[[1024,14],[-5473486921211236216,-6230876016761372746,3953448761157777955,4246796580750024372],[]]|
|2007-09-20 21:50:00.0000000|[[1024,14],[4835649640695509390],[]]|
|2007-09-29 08:10:00.0000000|[[1024,14],[4246796580750024372],[]]|
|2007-12-30 16:00:00.0000000|[[1024,14],[4246796580750024372,-8936707700542868125],[]]|