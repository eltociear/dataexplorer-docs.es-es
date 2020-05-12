---
title: 'format_datetime (): Explorador de datos de Azure'
description: En este artículo se describe format_datetime () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 073bf45977648bd654f72fff47b62f92ac1b3d27
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227390"
---
# <a name="format_datetime"></a>format_datetime()

Da formato a un valor de fecha y hora según el formato proporcionado.

```kusto
format_datetime(datetime(2015-12-14 02:03:04.12345), 'y-M-d h:m:s.fffffff') == "15-12-14 2:3:4.1234500"
```

**Sintaxis**

`format_datetime(`*fecha y hora* `,` *formato*`)`

**Argumentos**

* `datetime`: valor de un tipo `datetime` .
* `format`: especificador de formato cadena, que consta de uno o varios [elementos de formato](#supported-formats).

**Devuelve**

Cadena con el resultado de formato.

## <a name="supported-formats"></a>Formatos compatibles

|Especificador de formato   |Descripción    |Ejemplos
|---|---|---
|`d`    |El día del mes, de 1 a 31. | 2009-06-01T13:45:30-> 1, 2009-06-15T13:45:30-> 15
|`dd`   |El día del mes, de 01 a 31.| 2009-06-01T13:45:30-> 01, 2009-06-15T13:45:30-> 15
|`f`    |Las décimas de segundo de un valor de fecha y hora. |2009-06-15T13:45:30.6170000-> 6, 2009-06-15T13:45:30.05-> 0
|`ff`   |Las centésimas de segundo de un valor de fecha y hora. |2009-06-15T13:45:30.6170000-> 61, 2009-06-15T13:45:30.0050000-> 00
|`fff`  |Los milisegundos de un valor de fecha y hora. |6/15/2009 13:45:30.617-> 617, 6/15/2009 13:45:30.0005-> 000
|`ffff` |Las diezmilésimas de segundo de un valor de fecha y hora. |2009-06-15T13:45:30.6175000-> 6175, 2009-06-15T13:45:30.0000500-> 0000
|`fffff`    |Las cienmilésimas de segundo de un valor de fecha y hora. |2009-06-15T13:45:30.6175400-> 61754, 2009-06-15T13:45:30.000005-> 00000
|`ffffff`   |Las millonésimas de segundo de un valor de fecha y hora. |2009-06-15T13:45:30.6175420-> 617542, 2009-06-15T13:45:30.0000050-> 000000
|`fffffff`  |Las diezmillonésimas de segundo de un valor de fecha y hora. |2009-06-15T13:45:30.6175425-> 6175425, 2009-06-15T13:45:30.0001150-> 0001150
|`F`    |Si es distinto de cero, las décimas de segundo de un valor de fecha y hora. |2009-06-15T13:45:30.6170000-> 6, 2009-06-15T13:45:30.0500000-> (sin salida)
|`FF`   |Si es distinto de cero, las centésimas de segundo de un valor de fecha y hora. |2009-06-15T13:45:30.6170000-> 61, 2009-06-15T13:45:30.0050000-> (sin salida)
|`FFF`  |Si es distinto de cero, los milisegundos de un valor de fecha y hora. |2009-06-15T13:45:30.6170000-> 617, 2009-06-15T13:45:30.0005000-> (sin salida)
|`FFFF` |Si es distinto de cero, las diezmilésimas de segundo de un valor de fecha y hora. |2009-06-15T13:45:30.5275000-> 5275, 2009-06-15T13:45:30.0000500-> (sin salida)
|`FFFFF`    |Si es distinto de cero, las cienmilésimas de segundo de un valor de fecha y hora. |2009-06-15T13:45:30.6175400-> 61754, 2009-06-15T13:45:30.0000050-> (sin salida)
|`FFFFFF`   |Si es distinto de cero, las millonésimas de segundo de un valor de fecha y hora. |2009-06-15T13:45:30.6175420-> 617542, 2009-06-15T13:45:30.0000050-> (sin salida)
|`FFFFFFF`  |Si es distinto de cero, las diezmillonésimas de segundo de un valor de fecha y hora. |2009-06-15T13:45:30.6175425-> 6175425, 2009-06-15T13:45:30.0001150-> 000115
|`h`    |La hora, usando un reloj de 12 horas de 1 a 12. |2009-06-15T01:45:30-> 1, 2009-06-15T13:45:30-> 1
|`hh`   |La hora, usando un reloj de 12 horas de 01 a 12. |2009-06-15T01:45:30-> 01, 2009-06-15T13:45:30-> 01
|`H`    |La hora, usando un reloj de 24 horas de 0 a 23. |2009-06-15T01:45:30-> 1, 2009-06-15T13:45:30-> 13
|`HH`   |La hora, usando un reloj de 24 horas de 00 a 23. |2009-06-15T01:45:30-> 01, 2009-06-15T13:45:30-> 13
|`m`    |Minutos, de 0 a 59. |2009-06-15T01:09:30-> 9, 2009-06-15T13:29:30-> 29
|`mm`   |El minuto, de 00 a 59. |2009-06-15T01:09:30-> 09, 2009-06-15T01:45:30-> 45
|`M`    |El mes, de 1 a 12. |2009-06-15T13:45:30 -> 6
|`MM`   |El mes, de 01 a 12.|2009-06-15T13:45:30 -> 06
|`s`    |El segundo, de 0 a 59. |2009-06-15T13:45:09 -> 9
|`ss`   |El segundo, de 00 a 59. |2009-06-15T13:45:09 -> 09
|`y`    |El año, de 0 a 99. |0001-01-01T00:00:00-> 1, 0900-01-01T00:00:00-> 0, 1900-01-01T00:00:00-> 0, 2009-06-15T13:45:30-> 9, 2019-06-15T13:45:30-> 19
|`yy`   |El año, de 00 a 99. | 0001-01-01T00:00:00-> 01, 0900-01-01T00:00:00-> 00, 1900-01-01T00:00:00-> 00, 2019-06-15T13:45:30-> 19
|`yyyy` |El año como un número de cuatro dígitos. | 0001-01-01T00:00:00-> 0001, 0900-01-01T00:00:00-> 0900, 1900-01-01T00:00:00-> 1900, 2009-06-15T13:45:30-> 2009
|`tt`   |Horas de AM/PM |2009-06-15T13:45:09-> PM

**Delimitadores admitido**

El especificador de formato puede incluir los siguientes caracteres delimitadores:

|Delimitador|Comentario|
|---------|-------|
|`' '`| Space|
|`'/'`||
|`'-'`|Guión|
|`':'`||
|`','`||
|`'.'`||
|`'_'`||
|`'['`||
|`']'`||

**Ejemplos**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
let dt = datetime(2017-01-29 09:00:05);
print 
v1=format_datetime(dt,'yy-MM-dd [HH:mm:ss]'), 
v2=format_datetime(dt, 'yyyy-M-dd [H:mm:ss]'),
v3=format_datetime(dt, 'yy-MM-dd [hh:mm:ss tt]')
```

|v1|v2|v3|
|---|---|---|
|17-01-29 [09:00:05]|2017-1-29 [9:00:05]|17-01-29 [09:00:05 AM]|
