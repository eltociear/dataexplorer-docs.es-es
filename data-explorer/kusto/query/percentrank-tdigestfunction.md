---
title: 'percentrank_tdigest (): Explorador de datos de Azure'
description: En este artículo se describe percentrank_tdigest () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/10/2019
ms.openlocfilehash: 8220c52b70eec8a0a297c5826fff3a6e2a0483b3
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373236"
---
# <a name="percentrank_tdigest"></a>percentrank_tdigest()

Calcula el rango aproximado del valor en un conjunto donde el rango se expresa como porcentaje del tamaño del conjunto. Esta función se puede ver como la inversa del percentil.

**Sintaxis**

`percentrank_tdigest(`*TDigest* `,` *Expr*`)`

**Argumentos**

* *TDigest*: expresión generada por [TDigest ()](tdigest-aggfunction.md) o [tdigest_merge ()](tdigest-merge-aggfunction.md).
* *Expr*: expresión que representa un valor que se va a usar para el cálculo del porcentaje de clasificación.

**Devuelve**

El rango porcentual del valor de un conjunto de valores.

**Sugerencias**

1) El tipo de segundo parámetro y el tipo de los elementos de tdigest deben ser el mismo.

2) El primer parámetro debe ser TDigest generado por [TDigest ()](tdigest-aggfunction.md) o [tdigest_merge ()](tdigest-merge-aggfunction.md)

**Ejemplos**

Al obtener el percentrank_tdigest () de la propiedad Damage que tiene un valor de $4490 es ~ 85%:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty)
| project percentrank_tdigest(tdigestRes, 4490)

```

|Column1|
|---|
|85.0015237192293|


El uso de percentil 85 sobre la propiedad de daños debe proporcionar $4490:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty)
| project percentile_tdigest(tdigestRes, 85, typeof(long))

```

|percentile_tdigest_tdigestRes|
|---|
|4490|
