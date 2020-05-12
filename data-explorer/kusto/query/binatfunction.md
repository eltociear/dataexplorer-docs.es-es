---
title: 'bin_at (): Explorador de datos de Azure'
description: En este artículo se describe bin_at () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 90055f644dbf653eb65546202832f7cab834a0ac
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227608"
---
# <a name="bin_at"></a>bin_at()

Redondea los valores a un "bin" de tamaño fijo, con control sobre el punto inicial de la ubicación.
(Vea también [`bin function`](./binfunction.md) ).

**Sintaxis**

`bin_at``(` *Expresión* `,` *BinSize* `, ` *FixedPoint*`)`

**Argumentos**

* *Expresión*: expresión escalar de un tipo numérico (incluidos `datetime` y `timespan` ) que indica el valor que se va a redondear.
* *Bine*: una constante escalar del mismo tipo que la *expresión* que indica el tamaño de cada bin. 
* *FixedPoint*: una constante escalar del mismo tipo que la *expresión* que indica un valor de *expresión* que es un "punto fijo" (es decir, un valor `fixed_point` para el que `bin_at(fixed_point, bin_size, fixed_point) == fixed_point` ).

**Devuelve**

El *múltiplo más cercano de la* *expresión*de la parte inferior se ha desplazado para que *FixedPoint* se traduzca en sí mismo.

**Ejemplos**

|Expresión                                                                    |Resultado                           |Comentarios                   |
|------------------------------------------------------------------------------|---------------------------------|---------------------------|
|`bin_at(6.5, 2.5, 7)`                                                         |`4.5`                            ||
|`bin_at(time(1h), 1d, 12h)`                                                   |`-12h`                           ||
|`bin_at(datetime(2017-05-15 10:20:00.0), 1d, datetime(1970-01-01 12:00:00.0))`|`datetime(2017-05-14 12:00:00.0)`|Todas las ubicaciones estarán a mediodía   |
|`bin_at(datetime(2017-05-17 10:20:00.0), 7d, datetime(2017-06-04 00:00:00.0))`|`datetime(2017-05-14 00:00:00.0)`|Todas las ubicaciones estarán en domingo|


En el ejemplo siguiente, observe que el `"fixed point"` argumento Arg se devuelve como una de las ubicaciones y que las demás ubicaciones se alinean en función de `bin_size` . Tenga en cuenta también que cada bin DateTime representa la hora de inicio de la ubicación:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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
