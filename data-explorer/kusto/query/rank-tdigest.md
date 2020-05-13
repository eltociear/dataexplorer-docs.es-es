---
title: 'rank_tdigest (): Explorador de datos de Azure'
description: En este artículo se describe rank_tdigest () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/10/2019
ms.openlocfilehash: 29b35e5bd7265d89e65fe0129317a9f1672c7cad
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373077"
---
# <a name="rank_tdigest"></a>rank_tdigest()

Calcula el rango aproximado del valor de un conjunto. El rango de valores de `v` un conjunto `S` se define como el recuento de miembros de `S` menor o igual que `v` , `S` se representa mediante su `tdigest` .

**Sintaxis**

`rank_tdigest(`*`TDigest`*`,` *`Expr`*`)`

**Argumentos**

* *TDigest*: expresión generada por [TDigest ()](tdigest-aggfunction.md) o [tdigest_merge ()](tdigest-merge-aggfunction.md)
* *Expr*: expresión que representa un valor que se va a usar para el cálculo de la clasificación.

**Devuelve**

El valor foreach de rango en un conjunto de datos.

**Sugerencias**

1) Los valores que desea obtener su rango deben ser del mismo tipo que `tdigest` .

**Ejemplos**

En una lista ordenada (1-1000), el rango de 685 es su índice:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range x from 1 to 1000 step 1
| summarize t_x=tdigest(x)
| project rank_of_685=rank_tdigest(t_x, 685)
```

|`rank_of_685`|
|-------------|
|`685`        |

Esta consulta calcula el rango del valor $4490 en todas las propiedades de daños costos:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty)
| project rank_of_4490=rank_tdigest(tdigestRes, 4490) 

```

|`rank_of_4490`|
|--------------|
|`50207`       |

Obtener el porcentaje estimado del rango (dividiendo por el tamaño del conjunto):

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty), count()
| project rank_tdigest(tdigestRes, 4490) * 100.0 / count_

```

|`Column1`         |
|------------------|
|`85.0015237192293`|


El percentil 85 del costo de las propiedades de daños es $4490:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty)
| project percentile_tdigest(tdigestRes, 85, typeof(long))

```

|`percentile_tdigest_tdigestRes`|
|-------------------------------|
|`4490`                         |


