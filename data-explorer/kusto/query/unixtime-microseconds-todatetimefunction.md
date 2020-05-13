---
title: 'unixtime_microseconds_todatetime (): Explorador de datos de Azure'
description: En este artículo se describe unixtime_microseconds_todatetime () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/27/2019
ms.openlocfilehash: f69261e7ac84d8ae77fd1d2fb89f31ede2536443
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83370244"
---
# <a name="unixtime_microseconds_todatetime"></a>unixtime_microseconds_todatetime()

Convierte los microsegundos del tiempo de UNIX a una fecha y hora UTC.

**Sintaxis**

`unixtime_microseconds_todatetime(*microseconds*)`

**Argumentos**

* *microsegundos*: un número real representa la marca de tiempo temporal en microsegundos. `Datetime`Esto sucede antes de que el tiempo de tiempo (1970-01-01 00:00:00) tenga un valor de marca de tiempo negativo.

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un valor [DateTime](./scalar-data-types/datetime.md) . Si la conversión no se realiza correctamente, el resultado será null.

**Vea también**

* Convertir los segundos de tiempo de la hora de UNIX a DateTime con [unixtime_seconds_todatetime ()](unixtime-seconds-todatetimefunction.md).
* Convierta los milisegundos de la época de UNIX a fecha y hora UTC mediante [unixtime_milliseconds_todatetime ()](unixtime-milliseconds-todatetimefunction.md).
* Convertir nanosegundos de tiempo de UNIX a DateTime de UTC mediante [unixtime_nanoseconds_todatetime ()](unixtime-nanoseconds-todatetimefunction.md).

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples  -->
```kusto
print date_time = unixtime_microseconds_todatetime(1546300800000000)
```

|date_time|
|---|
|2019-01-01 00:00:00.0000000|
