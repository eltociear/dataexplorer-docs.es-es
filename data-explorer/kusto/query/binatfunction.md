---
title: bin_at() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe bin_at() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 218142e1c377e6a72abde154d4576025698b2f0d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517449"
---
# <a name="bin_at"></a>bin_at()

Redondea los valores hacia abajo a una "bin" de tamaño fijo, con control sobre el punto inicial de la ubicación.
(Véase [`bin function`](./binfunction.md)también .)

**Sintaxis**

`bin_at``(` *Expression* Expresión`,` *BinSize* `, ` *FixedPoint*`)`

**Argumentos**

* *Expresión*: Una expresión escalar de un `datetime` `timespan`tipo numérico (incluyendo y ) que indica el valor a redondear.
* *BinSize*: Una constante escalar del mismo tipo que *Expression* que indica el tamaño de cada bin. 
* *FixedPoint*: Una constante escalar del mismo tipo que *Expresión* que indica un valor de *Expresión* `fixed_point` que `bin_at(fixed_point, bin_size, fixed_point) == fixed_point`es un "punto fijo" (es decir, un valor para el que .)

**Devuelve**

El múltiplo más cercano de *BinSize* por debajo de *Expresión*, se ha desplazado para que *FixedPoint* se traduzca a sí mismo.

**Ejemplos**

|Expression                                                                    |Resultado                           |Comentarios                   |
|------------------------------------------------------------------------------|---------------------------------|---------------------------|
|`bin_at(6.5, 2.5, 7)`                                                         |`4.5`                            ||
|`bin_at(time(1h), 1d, 12h)`                                                   |`-12h`                           ||
|`bin_at(datetime(2017-05-15 10:20:00.0), 1d, datetime(1970-01-01 12:00:00.0))`|`datetime(2017-05-14 12:00:00.0)`|Todos los contenedores estarán al mediodía   |
|`bin_at(datetime(2017-05-17 10:20:00.0), 7d, datetime(2017-06-04 00:00:00.0))`|`datetime(2017-05-14 00:00:00.0)`|Todos los contenedores serán los domingos|


En el ejemplo siguiente, `"fixed point"` observe que el arg se devuelve como una de las `bin_size`ubicaciones y las otras ubicaciones se alinean con él en función del archivo . Tenga en cuenta también que cada bin datetime representa la hora de inicio de esa ubicación:

```kusto

datatable(Date:datetime, Num:int)[
datetime(2018-02-24T15:14),3,
datetime(2018-02-23T16:14),4,
datetime(2018-02-26T15:14),5]
| summarize sum(Num) by bin_at(Date, 1d, datetime(2018-02-24 15:14:00.0000000)) 
```

|Date|sum_Num|
|---|---|
|2018-02-23 15:14:00.0000000|4|
|2018-02-24 15:14:00.0000000|3|
|2018-02-26 15:14:00.0000000|5|