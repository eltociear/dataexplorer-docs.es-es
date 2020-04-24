---
title: tdigest() (función de agregación) - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe tdigest() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/10/2019
ms.openlocfilehash: 6d36e33891aae2c1c4740d8190047bb4a71b3254
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506416"
---
# <a name="tdigest-aggregation-function"></a>tdigest() (función de agregación)

Calcula los resultados [`percentiles()`](percentiles-aggfunction.md) intermedios de todo el grupo. 

* Solo se puede utilizar en el contexto de la agregación dentro de [resumir](summarizeoperator.md).

Obtenga más información sobre el [algoritmo subyacente (T-Digest) y el error estimado.](percentiles-aggfunction.md#estimation-error-in-percentiles)

**Sintaxis**

`summarize``tdigest(` *Expr* `,` [ *WeightExpr*]`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. 
* *WeightExpr*: Expresión que se utilizará como peso de valores para el cálculo de la agregación.

    
**Devuelve**

Los resultados intermedios de los percentiles ponderados de *Expr* en todo el grupo.
 
 
**Sugerencias**

1) Puede utilizar la función de agregación [tdigest_merge()](tdigest-merge-aggfunction.md) para fusionar de nuevo la salida de tdigest en otro grupo.

2) Puede utilizar la función [percentile_tdigest()] (percentile-tdigestfunction.md) para calcular el percentil/percentil del porcentaje de los resultados de tdigest.

**Ejemplos**

```kusto
StormEvents
| summarize tdigest(DamageProperty) by State
```

|State|tdigest_DamageProperty|
|---|---|
|ATLANTIC SOUTH|[[5],[0],[193]]|
|Florida|[[5],[250,10,600000,5000,375000,15000000,20000,6000000,0,110000,150000,500,12000,30000,15000,46000000,7000000,6200000,200000,40000,8000,52000000,62000000,1200000,130000,1500000,4000000,7000,250000,875000,3000,100000,10600000,300000,1000000,25000,75000,2000,60000,10000,170000,350000,50000,1000,16000,80000,2500,400000],[9,1,1,22,1,1,9,1,842,1,3,7,2,4,7,1,1,1,2,5,3,3,1,1,1,1,2,2,1,1,9,7,1,1,2,5,2,9,2,27,1,1,7,27,1,1,1,1]]|
|Georgia|[[5],[468,209,300000,3000,250000,775000,14000,500000,0,75000,4500000,500,6928,22767,9714,800000,700000,600000,150000,25000,5000,1600000,1250000,2700000,1500000,2250000,400000,4000,175000,325000,2500,73750,750000,1400000,350000,28000000,39000,1500,35000,6455,140000,225000,30000,1000,110000000,21700000,2000,275000,200000,100000,1000000,2600000,370000,2100000,355000,117500,50000,20100,10000],[11,11,4,53,21,1,6,10,1317,8,1,56,8,6,7,1,1,1,14,29,69,1,2,1,1,1,3,14,5,1,3,4,4,1,4,1,5,14,3,5,2,1,9,96,1,1,72,1,10,17,3,1,1,1,1,2,21,4,31]]|
|Mississippi|[[5],[267,55,90000,3000,75000,300000,11167,160000,0,32000,40000,1000,7000,13000,8000,400000,200000,180000,50000,15000,5000,700000,500000,120000,650000,1000000,150000,4000,60000,100000,2500,30000,250000,600000,110000,12000,20000,1500,17000,6000,45000,70000,15250,1219,10000,25000,2000,80000,65000,35000,450000,1200000,130000,750000],[3,2,6,21,1,4,6,1,741,4,13,44,8,2,8,1,5,1,23,21,32,1,3,1,1,1,5,18,17,4,1,14,2,4,4,16,13,10,4,9,2,10,4,8,31,17,51,13,1,1,1,2,1,1]]|
|SAMOA AMERICANA|[[5],[0,250000],[15,1]]|