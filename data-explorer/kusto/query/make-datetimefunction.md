---
title: make_datetime() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe make_datetime() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: c51b99e4d668ec4a5f96cdfd72442d7bcab553a5
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512927"
---
# <a name="make_datetime"></a>make_datetime()

Crea un valor escalar [datetime](./scalar-data-types/datetime.md) a partir de la fecha y hora especificadas.

```kusto
make_datetime(2017,10,01,12,10) == datetime(2017-10-01 12:10)
```

**Sintaxis**

`make_datetime(`*año,**mes,**día*`)`

`make_datetime(`*año,**mes,**día,**hora,**minuto*`)`

`make_datetime(`*año,**mes,**día,**hora,**minuto,**segundo*`)`

**Argumentos**

* *año*: año (un valor entero, de 0 a 9999)
* *mes*: mes (un valor entero, de 1 a 12)
* *día*: día (un valor entero, de 1 a 28-31)
* *hora*: hora (un valor entero, de 0 a 23)
* *minuto*: minuto (un valor entero, de 0 a 59)
* *segundo:* segundo (un valor real, de 0 a 59.9999999)

**Devuelve**

Si la creación se realiza correctamente, el resultado será un valor [datetime,](./scalar-data-types/datetime.md) de lo contrario, result será null.
 
**Ejemplo**

```kusto
print year_month_day = make_datetime(2017,10,01)
```

|year_month_day|
|---|
|2017-10-01 00:00:00.0000000|




```kusto
print year_month_day_hour_minute = make_datetime(2017,10,01,12,10)
```

|year_month_day_hour_minute|
|---|
|2017-10-01 12:10:00.0000000|




```kusto
print year_month_day_hour_minute_second = make_datetime(2017,10,01,12,11,0.1234567)
```

|year_month_day_hour_minute_second|
|---|
|2017-10-01 12:11:00.1234567|

