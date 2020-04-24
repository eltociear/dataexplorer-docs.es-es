---
title: unixtime_milliseconds_todatetime() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe unixtime_milliseconds_todatetime() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/25/2019
ms.openlocfilehash: ab229d78d2a9ff5a7e50ecefe027824488578b12
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505260"
---
# <a name="unixtime_milliseconds_todatetime"></a>unixtime_milliseconds_todatetime()

Convierte milisegundos unix-epoch en fecha y hora UTC.

**Sintaxis**

`unixtime_milliseconds_todatetime(*milliseconds*)`

**Argumentos**

* *milisegundos*: Un número real representa la marca de tiempo de la época en milisegundos. `Datetime`que ocurre antes de la hora de la época (1970-01-01 00:00:00) tiene un valor de marca de tiempo negativo.

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un valor [datetime.](./scalar-data-types/datetime.md) Si la conversión no se realiza correctamente, el resultado será null.

**Vea también**

* Convierta unix-epoch seconds a UTC datetime usando [unixtime_seconds_todatetime()](unixtime-seconds-todatetimefunction.md).
* Convierta microsegundos unix-epoch a UTC datetime utilizando [unixtime_microseconds_todatetime()](unixtime-microseconds-todatetimefunction.md).
* Convierta nanosegundos unix-epoch a UTC datetime utilizando [unixtime_nanoseconds_todatetime()](unixtime-nanoseconds-todatetimefunction.md).

**Ejemplo**

```kusto
print date_time = unixtime_milliseconds_todatetime(1546300800000)
```

|date_time|
|---|
|2019-01-01 00:00:00.0000000|