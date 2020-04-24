---
title: bin() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe bin() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 3fb827c71fa63fde031a91bc9aec7f0ed108fd5c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517432"
---
# <a name="bin"></a>bin()

Redondea los valores hacia abajo hasta un entero múltiplo del tamaño de un intervalo determinado. 

Se utiliza con [`summarize by ...`](./summarizeoperator.md)frecuencia en combinación con .
Si tiene un conjunto de valores dispersos, se agruparán en un conjunto más pequeño de valores específicos.

Los valores nulos, un tamaño de ubicación nulo o un tamaño de ubicación negativo darán como resultado null. 

Alias `floor()` para funcionar.

**Sintaxis**

`bin(`*valor*`,`*roundTo*`)`

**Argumentos**

* *valor*: Un número, fecha o intervalo de tiempo. 
* *roundTo*: El "tamaño de la bandeja". Un número, una fecha o un intervalo de tiempo que divide *value*. 

**Devuelve**

El múltiplo más cercano de *roundTo* por debajo de *value*.  
 
```kusto
(toint((value/roundTo))) * roundTo`
```

**Ejemplos**

Expression | Resultado
---|---
`bin(4.5, 1)` | `4.0`
`bin(time(16d), 7d)` | `14d`
`bin(datetime(1970-05-11 13:45:07), 1d)`|  `datetime(1970-05-11)`


La siguiente expresión calcula un histograma de duraciones, con un tamaño de depósito de 1 segundo:

```kusto
T | summarize Hits=count() by bin(Duration, 1s)
```