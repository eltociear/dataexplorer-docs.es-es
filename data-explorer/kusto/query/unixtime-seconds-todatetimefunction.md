---
title: unixtime_seconds_todatetime() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe unixtime_seconds_todatetime() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/25/2019
ms.openlocfilehash: 74abf66dabb7a8fc74085c695081d6a11bfdf3a3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505226"
---
# <a name="unixtime_seconds_todatetime"></a>unixtime_seconds_todatetime()

Convierte segundos unix-epoch en UTC datetime.

**Sintaxis**

`unixtime_seconds_todatetime(*seconds*)`

**Argumentos**

* *segundos*: Un número real representa la marca de tiempo de la época en segundos. `Datetime`que ocurre antes de la hora de la época (1970-01-01 00:00:00) tiene un valor de marca de tiempo negativo.

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un valor [datetime.](./scalar-data-types/datetime.md) Si la conversión no se realiza correctamente, el resultado será null.

**Vea también**

* Convierta milisegundos unix-epoch a UTC datetime utilizando [unixtime_milliseconds_todatetime()](unixtime-milliseconds-todatetimefunction.md).
* Convierta microsegundos unix-epoch a UTC datetime utilizando [unixtime_microseconds_todatetime()](unixtime-microseconds-todatetimefunction.md).
* Convierta nanosegundos unix-epoch a UTC datetime utilizando [unixtime_nanoseconds_todatetime()](unixtime-nanoseconds-todatetimefunction.md).

**Ejemplo**

```kusto
print date_time = unixtime_seconds_todatetime(1546300800)
```

|date_time|
|---|
|2019-01-01 00:00:00.0000000|