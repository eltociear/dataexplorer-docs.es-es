---
title: datetime_part() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe datetime_part() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/18/2020
ms.openlocfilehash: c64208725f0d5c49a7ea7733f8eb5a208e19225b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516378"
---
# <a name="datetime_part"></a>datetime_part()

Extrae la parte de fecha solicitada como un valor entero.

```kusto
datetime_part("Day",datetime(2015-12-14))
```

**Sintaxis**

`datetime_part(`*parte*`,`*datetime*`)`

**Argumentos**

* `date`: `datetime`
* `part`: `string`

Posibles valores `part`de : 
- Year
- Trimestre
- Mes
- week_of_year
- Día
- DayOfYear
- Hour
- Minute
- Segundo
- Millisecond
- Microsegundo
- Nanosegundo

**Devuelve**

Un entero que representa la pieza extraída.

**Nota**

`week_of_year`devuelve un entero que representa el número de semana. El número de semana se calcula a partir de la primera semana de un año, que es la que incluye el primer jueves.

**Ejemplos**

```kusto
let dt = datetime(2017-10-30 01:02:03.7654321); 
print 
year = datetime_part("year", dt),
quarter = datetime_part("quarter", dt),
month = datetime_part("month", dt),
weekOfYear = datetime_part("week_of_year", dt),
day = datetime_part("day", dt),
dayOfYear = datetime_part("dayOfYear", dt),
hour = datetime_part("hour", dt),
minute = datetime_part("minute", dt),
second = datetime_part("second", dt),
millisecond = datetime_part("millisecond", dt),
microsecond = datetime_part("microsecond", dt),
nanosecond = datetime_part("nanosecond", dt)

```

|year|quarter|month|weekOfYear|day|dayOfYear|hour|minute|second|milisegundo|microsegundo|nanosegundo|
|---|---|---|---|---|---|---|---|---|---|---|---|
|2017|4|10|44|30|303|1|2|3|765|765432|765432100|

> [!NOTE]
> `weekofyear`es una variante `week_of_year` obsoleta de la parte. `weekofyear`no era compatible con la normas ISO 8601; la primera semana de un año se definió como la semana con el primer miércoles del año en ella.
`week_of_year`es compatible con la normas ISO 8601; la primera semana de un año se define como la semana con el primer jueves del año en ella. [Para obtener más información](https://en.wikipedia.org/wiki/ISO_8601#Week_dates).