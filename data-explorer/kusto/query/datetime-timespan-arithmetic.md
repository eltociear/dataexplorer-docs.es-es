---
title: Fecha y hora / aritmética de intervalo de tiempo - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe la aritmética de fecha y hora y intervalo de tiempo en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/27/2019
ms.openlocfilehash: 34bda7b8418dd519995f25c625a5031202a64a8b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516327"
---
# <a name="datetime--timespan-arithmetic"></a>Fecha y hora / aritmética de intervalo de tiempo

Kusto admite la realización de `datetime` `timespan`operaciones aritméticas en valores de tipos y:

* Se pueden restar (pero `datetime` no agregar) dos valores para obtener un `timespan` valor que exprese su diferencia.
  Por ejemplo, `datetime(1997-06-25) - datetime(1910-06-11)` es la edad de [Jacques-Yves Cousteau](https://en.wikipedia.org/wiki/Jacques_Cousteau) cuando murió.

* Se pueden sumar `timespan` o restar dos valores para obtener un `timespan` valor que es su suma o diferencia.
  Por ejemplo, `1d + 2d` es de tres días.

* Se puede agregar `timespan` o restar un valor de un `datetime` valor.
  Por ejemplo, `datetime(1910-06-11) + 1d` es la fecha en que Cousteau se volvió un día de edad.

* Uno puede `timespan` dividir dos valores para obtener su cociente.
  Por `1d / 5h` ejemplo, `4.8`da .
  Esto le da a `timespan` uno la capacidad `timespan` de expresar cualquier valor como un múltiplo de otro valor. Por ejemplo, para expresar una hora `1h` `1s`en `1h / 1s` segundos, simplemente `3600`divida por : (con el resultado obvio, ).

* Por el contrario, se puede múlparte un valor numérico `double` (como y `long`) por un `timespan` valor para obtener un `timespan` valor.
  Por ejemplo, se puede expresar una `1.5 * 1h`hora y media como .

## <a name="example-unix-time"></a>Ejemplo: Tiempo Unix

[La hora Unix](https://en.wikipedia.org/wiki/Unix_time) (también conocida como hora POSIX o hora de la época DE UNIX) es un sistema para describir un punto en el tiempo como el número de segundos que han transcurrido desde las 00:00:00 Jueves, 1 de enero de 1970, Hora universal coordinada (UTC), menos segundos bisiestos.

Si los datos incluyen la representación del tiempo Unix como un entero, o si necesita convertirlo en él, están disponibles las siguientes funciones:

```kusto
let fromUnixTime = (t:long)
{ 
    datetime(1970-01-01) + t * 1sec 
};
print result = fromUnixTime(1546897531)
```

|resultado                     |
|---------------------------|
|2019-01-07 21:45:31.0000000|

```kusto
let toUnixTime = (dt:datetime) 
{ 
    (dt - datetime(1970-01-01)) / 1s 
};
print result = toUnixTime(datetime(2019-01-07 21:45:31.0000000))
```

|resultado                     |
|---------------------------|
|1546897531                 |

> [!NOTE]
> Además de las funciones anteriores, véase funciones dedicadas para conversiones de tiempo unix-epoch: [unixtime_seconds_todatetime()](unixtime-seconds-todatetimefunction.md)
> [unixtime_milliseconds_todatetime()](unixtime-milliseconds-todatetimefunction.md)
> [unixtime_microseconds_todatetime()](unixtime-microseconds-todatetimefunction.md)
> [unixtime_nanoseconds_todatetime()](unixtime-nanoseconds-todatetimefunction.md)