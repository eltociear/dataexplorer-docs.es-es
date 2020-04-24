---
title: percentrank_tdigest() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe percentrank_tdigest() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/10/2019
ms.openlocfilehash: fe356ddb2e6301bbb283d2e6aa59b5c98f8bf3fe
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511193"
---
# <a name="percentrank_tdigest"></a>percentrank_tdigest()

Calcula el rango aproximado del valor en un conjunto donde el rango se expresa como porcentaje del tamaño del conjunto. Esta función se puede ver como la inversa del percentil.

**Sintaxis**

`percentrank_tdigest(`*TDigest* `,` *Expr*`)`

**Argumentos**

* *TDigest*: Expresión generada por [tdigest()](tdigest-aggfunction.md) o [tdigest_merge()](tdigest-merge-aggfunction.md).
* *Expr*: Expresión que representa un valor que se utilizará para el cálculo de la clasificación porcentual.

**Devuelve**

El rango porcentual del valor en un conjunto de datos.

**Sugerencias**

1) El tipo de segundo parámetro y el tipo de los elementos en el tdigest deben ser los mismos.

2) El primer parámetro debe ser TDigest generado por [tdigest()](tdigest-aggfunction.md) o [tdigest_merge()](tdigest-merge-aggfunction.md)

**Ejemplos**

Obtener el percentrank_tdigest() de la propiedad de daños que valoró 4490$ es del 85%:

```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty)
| project percentrank_tdigest(tdigestRes, 4490)

```

|Column1|
|---|
|85.0015237192293|


El uso del percentil 85 sobre la propiedad de daños debe dar 4490$:

```kusto
StormEvents
| summarize tdigestRes = tdigest(DamageProperty)
| project percentile_tdigest(tdigestRes, 85, typeof(long))

```

|percentile_tdigest_tdigestRes|
|---|
|4490|