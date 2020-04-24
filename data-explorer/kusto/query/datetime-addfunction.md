---
title: datetime_add() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe datetime_add() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: ead7e0ae5c4dee94930afe1b20c4d5b99e2b4664
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516463"
---
# <a name="datetime_add"></a>datetime_add()

Calcula una nueva [fecha y hora](./scalar-data-types/datetime.md) a partir de un datepart especificado multiplicado por una cantidad especificada, agregada a una fecha y [hora](./scalar-data-types/datetime.md)especificada.

**Sintaxis**

`datetime_add(`*cantidad*`,`*del*`,`período*datetime*`)`

**Argumentos**

* `period`: [cadena](./scalar-data-types/string.md). 
* `amount`: [entero](./scalar-data-types/int.md).
* `datetime`: valor [datetime.](./scalar-data-types/datetime.md)

Posibles valores de *período:* 
- Year
- Trimestre
- Mes
- Semana
- Día
- Hour
- Minute
- Segundo
- Millisecond
- Microsegundo
- Nanosegundo

**Devuelve**

Se ha añadido una fecha después de un intervalo de fecha/hora determinado.

**Ejemplos**

```kusto
print  year = datetime_add('year',1,make_datetime(2017,1,1)),
quarter = datetime_add('quarter',1,make_datetime(2017,1,1)),
month = datetime_add('month',1,make_datetime(2017,1,1)),
week = datetime_add('week',1,make_datetime(2017,1,1)),
day = datetime_add('day',1,make_datetime(2017,1,1)),
hour = datetime_add('hour',1,make_datetime(2017,1,1)),
minute = datetime_add('minute',1,make_datetime(2017,1,1)),
second = datetime_add('second',1,make_datetime(2017,1,1))

```

|year|quarter|month|week|day|hour|minute|second|
|---|---|---|---|---|---|---|---|
|2018-01-01 00:00:00.0000000|2017-04-01 00:00:00.0000000|2017-02-01 00:00:00.0000000|2017-01-08 00:00:00.0000000|2017-01-02 00:00:00.0000000|2017-01-01 01:00:00.0000000|2017-01-01 00:01:00.0000000|2017-01-01 00:00:01.0000000|






