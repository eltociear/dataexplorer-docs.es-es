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
ms.openlocfilehash: a849cd496d41ad473768b3f267639eaf8c467880
ms.sourcegitcommit: 4f68d6dbfa6463dbb284de0aa17fc193d529ce3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82741778"
---
# <a name="rank_tdigest"></a>rank_tdigest()

Calcula el rango aproximado del valor de un conjunto. El rango de `v` valores de un `S` conjunto se define como el recuento `S` de miembros de menor o igual `v`que `S` , se representa mediante `tdigest`su.

**Sintaxis**

`rank_tdigest(`*`TDigest`*`,` *`Expr`*`)`

**Argumentos**

* *TDigest*: expresión generada por [TDigest ()](tdigest-aggfunction.md) o [tdigest_merge ()](tdigest-merge-aggfunction.md)
* *Expr*: expresión que representa un valor que se va a usar para el cálculo de la clasificación.

**Devuelve**

El valor foreach de rango en un conjunto de datos.

**Sugerencias**

1) Los valores que desea obtener su rango deben ser del mismo tipo que `tdigest`.

**Ejemplos**

En una lista ordenada (1-1000), el rango de 685 es su índice:

```kusto
range x from 1 to 1000 step 1
| summarize t_x=tdigest(x)
| project rank_of_685=rank_tdigest(t_x, 685)
```

|`rank_of_685`|
|-------------|
|`685`        |

Esta consulta calcula el rango del valor $4490 en todas las propiedades de daños costos:

```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty)
| project rank_of_4490=rank_tdigest(tdigestRes, 4490) 

```

|`rank_of_4490`|
|--------------|
|`50207`       |

Obtener el porcentaje estimado del rango (dividiendo por el tamaño del conjunto):

```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty), count()
| project rank_tdigest(tdigestRes, 4490) * 100.0 / count_

```

|`Column1`         |
|------------------|
|`85.0015237192293`|


El percentil 85 del costo de las propiedades de daños es $4490:

```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty)
| project percentile_tdigest(tdigestRes, 85, typeof(long))

```

|`percentile_tdigest_tdigestRes`|
|-------------------------------|
|`4490`                         |


