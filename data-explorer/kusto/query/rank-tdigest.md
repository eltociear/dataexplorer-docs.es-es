---
title: rank_tdigest() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe rank_tdigest() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/10/2019
ms.openlocfilehash: ea24213b0ca673c39f399c3a12cc54cd7d7f47d5
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510547"
---
# <a name="rank_tdigest"></a>rank_tdigest()

Calcula el rango aproximado del valor en un conjunto. El rango `v` de `S` valor de un conjunto `S` se define como `v` `S` el recuento de miembros que son más pequeños o iguales a , se representa mediante su tdigest.

**Sintaxis**

`rank_tdigest(`*TDigest* `,` *Expr*`)`

**Argumentos**

* *TDigest*: Expresión generada por [tdigest()](tdigest-aggfunction.md) o [tdigest_merge()](tdigest-merge-aggfunction.md)
* *Expr*: Expresión que representa un valor que se utilizará para el cálculo de clasificación.

**Devuelve**

El rango foreach valor en un conjunto de datos.

**Sugerencias**

1) Los valores que desea obtener su rango deben ser del mismo tipo que el tdigest.

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

Esta consulta calcula el rango de valor 4490$ sobre todos los costos de las propiedades de daño:

```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty)
| project rank_of_4490=rank_tdigest(tdigestRes, 4490) 

```

|`rank_of_4490`|
|--------------|
|`50207`       |

Obtención del porcentaje estimado del rango (dividiendo por el tamaño establecido):

```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty), count()
| project rank_tdigest(tdigestRes, 4490) * 100.0 / count_

```

|`Column1`         |
|------------------|
|`85.0015237192293`|


El percentil 85 de los costos de las propiedades de daños es 4490$ :

```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty)
| project percentile_tdigest(tdigestRes, 85, typeof(long))

```

|`percentile_tdigest_tdigestRes`|
|-------------------------------|
|`4490`                         |


