---
title: 'hash (): Explorador de datos de Azure'
description: En este artículo se describe el hash () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a44f817ea57a114400f45e9ca2a841150b4590a6
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226812"
---
# <a name="hash"></a>hash()

Devuelve un valor hash para el valor de entrada.

**Sintaxis**

`hash(`*origen* [ `,` *mod*]`)`

**Argumentos**

* *origen*: el valor al que se va a aplicar un algoritmo hash.
* *mod*: valor de módulo opcional que se va a aplicar al resultado hash, de modo que el valor de salida esté comprendido entre `0` y *mod* -1

**Devuelve**

Valor hash del valor escalar dado, módulo el valor de mod especificado (si se especifica).

> [!WARNING]
> El algoritmo usado para calcular el hash es xxhash.
> Este algoritmo podría cambiar en el futuro y la única garantía es que, dentro de una sola consulta, todas las invocaciones de este método usan el mismo algoritmo.
> Por lo tanto, se recomienda a los usuarios que no almacenen los resultados de `hash()` en una tabla. Si se requieren valores hash persistentes, considere la posibilidad de usar [hash_sha256 ()](./sha256hashfunction.md) en su lugar (pero tenga en cuenta que es mucho más complejo calcular que `hash()` ).

**Ejemplos**

```kusto
hash("World")                   // 1846988464401551951
hash("World", 100)              // 51 (1846988464401551951 % 100)
hash(datetime("2015-01-01"))    // 1380966698541616202
```

En el ejemplo siguiente se usa la función hash para ejecutar una consulta en el 10% de los datos, resulta útil usar la función hash para el muestreo de los datos cuando se supone que el valor se distribuye uniformemente (en este valor StartTime de ejemplo)

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where hash(StartTime, 10) == 0
| summarize StormCount = count(), TypeOfStorms = dcount(EventType) by State 
| top 5 by StormCount desc
```
