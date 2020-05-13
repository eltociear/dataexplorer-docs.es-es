---
title: 'unixtime_milliseconds_todatetime (): Explorador de datos de Azure'
description: En este artículo se describe unixtime_milliseconds_todatetime () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/25/2019
ms.openlocfilehash: 26f1eb901798a28996c8cdc148fe68a71fb9d2b8
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83370275"
---
# <a name="unixtime_milliseconds_todatetime"></a>unixtime_milliseconds_todatetime()

Convierte los milisegundos del tiempo de la época de UNIX en fecha y hora UTC.

**Sintaxis**

`unixtime_milliseconds_todatetime(*milliseconds*)`

**Argumentos**

* *milisegundos*: un número real representa la marca de tiempo temporal en milisegundos. `Datetime`Esto sucede antes de que el tiempo de tiempo (1970-01-01 00:00:00) tenga un valor de marca de tiempo negativo.

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un valor [DateTime](./scalar-data-types/datetime.md) . Si la conversión no se realiza correctamente, el resultado será null.

**Vea también**

* Convertir los segundos de tiempo de la hora de UNIX a DateTime con [unixtime_seconds_todatetime ()](unixtime-seconds-todatetimefunction.md).
* Convertir microsegundos de tiempo de UNIX a fecha y hora UTC mediante [unixtime_microseconds_todatetime ()](unixtime-microseconds-todatetimefunction.md).
* Convertir nanosegundos de tiempo de UNIX a DateTime de UTC mediante [unixtime_nanoseconds_todatetime ()](unixtime-nanoseconds-todatetimefunction.md).

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples  -->
```kusto
print date_time = unixtime_milliseconds_todatetime(1546300800000)
```

|date_time|
|---|
|2019-01-01 00:00:00.0000000|
