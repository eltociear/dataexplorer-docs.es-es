---
title: hash() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe hash() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: f8142c42dcb0874dfbd84515e56dc8765bcba3d7
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514151"
---
# <a name="hash"></a>hash()

Devuelve un valor hash para el valor de entrada.

**Sintaxis**

`hash(`*fuente* `,` [ *mod*]`)`

**Argumentos**

* *source*: El valor que se va a aplicar un algoritmo hash.
* *mod*: Un valor de módulo opcional que se aplicará al `0` resultado hash, de modo que el valor de salida esté entre y *mod* - 1

**Devuelve**

El valor hash del escalar dado, modulo el valor mod dado (si se especifica).

> [!WARNING]
> El algoritmo utilizado para calcular el hash es xxhash.
> Este algoritmo puede cambiar en el futuro, y la única garantía es que dentro de una sola consulta todas las invocaciones de este método utilizan el mismo algoritmo.
> Por lo tanto, se recomienda a `hash()` los usuarios que no almacenen los resultados de una tabla. Si se requieren valores hash persistentes, considere la posibilidad de utilizar [hash_sha256()](./sha256hashfunction.md) en su lugar (pero tenga en cuenta que es mucho más complejo calcular que `hash()`).

**Ejemplos**

```kusto
hash("World")                   // 1846988464401551951
hash("World", 100)              // 51 (1846988464401551951 % 100)
hash(datetime("2015-01-01"))    // 1380966698541616202
```

En el ejemplo siguiente se utiliza la función hash para ejecutar una consulta en el 10% de los datos, es útil utilizar la función hash para muestrear los datos cuando se asume que el valor se distribuye uniformemente (en este ejemplo StartTime valor)

```kusto
StormEvents 
| where hash(StartTime, 10) == 0
| summarize StormCount = count(), TypeOfStorms = dcount(EventType) by State 
| top 5 by StormCount desc
```