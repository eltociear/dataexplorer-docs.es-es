---
title: Operadores numéricos- Explorador de datos de Azure Microsoft Docs
description: En este artículo se describen los operadores numéricos en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 6667005960152814be952d93fe932b4971d5322b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512026"
---
# <a name="numerical-operators"></a>Operadores numéricos

Los `int`tipos `long`, `real` , y representan tipos numéricos.
Los siguientes operadores se pueden utilizar entre pares de estos tipos:

Operator       |Descripción                         |Ejemplo
---------------|------------------------------------|-----------------------
`+`            |Sumar                                 |`3.14 + 3.14`, `ago(5m) + 5m`
`-`            |Restar                            |`0.23 - 0.22`,
`*`            |Multiplicar                            |`1s * 5`, `2 * 2`
`/`            |Divide                              |`10m / 1s`, `4 / 2`
`%`            |Módulo                              |`4 % 2`
`<`            |Menor que                                |`1 < 10`, `10sec < 1h`, `now() < datetime(2100-01-01)`
`>`            |Mayor que                             |`0.23 > 0.22`, `10min > 1sec`, `now() > ago(1d)`
`==`           |Equals                              |`1 == 1`
`!=`           |Not Equals                          |`1 != 0`
`<=`           |Menor o igual                       |`4 <= 5`
`>=`           |Mayor o igual                    |`5 >= 4`
`in`           |Igual a uno de los elementos       |[ver aquí](inoperator.md)
`!in`          |No es igual a uno de los elementos   |[ver aquí](inoperator.md)

**Comentario sobre el operador de módulo**

El módulo de dos números siempre devuelve en Kusto un "pequeño número no negativo".
Por lo tanto, el módulo de dos números, &le; *N* % *D*, es tal que: 0 (*N* % *D*) &lt; abs(*D*).

Por ejemplo, la consulta siguiente:

```kusto
print plusPlus = 14 % 12, minusPlus = -14 % 12, plusMinus = 14 % -12, minusMinus = -14 % -12
```

Produce este resultado:

|plusPlus  | minusPlus  | plusMinus  | minusMenos|
|----------|------------|------------|-----------|
|2         | 10         | 2          | 10        |