---
title: bin_auto() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe bin_auto() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: ebb214ae6a2676bf59a37e1e4e9cc3c085374bb3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517840"
---
# <a name="bin_auto"></a>bin_auto()

Redondea los valores a una "bin" de tamaño fijo, con control sobre el tamaño de ubicación y el punto de partida proporcionados por una propiedad de consulta.

**Sintaxis**

`bin_auto``(` *Expresión*`)`

**Argumentos**

* *Expresión*: Una expresión escalar de un tipo numérico que indica el valor a redondear.

**Propiedades de solicitud de cliente**

* `query_bin_auto_size`: literal numérico que indica el tamaño de cada ubicación.
* `query_bin_auto_at`: un literal numérico que indica un valor de *Expresión* que es `fixed_point` un `bin_auto(fixed_point)` == `fixed_point`"punto fijo" (es decir, un valor para el que .)

**Devuelve**

El múltiplo más cercano de `query_bin_auto_at` *la siguiente Expresión,* desplazado para que `query_bin_auto_at` se traduzca a sí mismo.

**Ejemplos**

```kusto
set query_bin_auto_size=1h;
set query_bin_auto_at=datetime(2017-01-01 00:05);
range Timestamp from datetime(2017-01-01 00:05) to datetime(2017-01-01 02:00) step 1m
| summarize count() by bin_auto(Timestamp)
```

|Timestamp                    | count_|
|-----------------------------|-------|
|2017-01-01 00:05:00.0000000  | 60    |
|2017-01-01 01:05:00.0000000  | 56    |