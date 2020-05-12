---
title: 'format_timespan (): Explorador de datos de Azure'
description: En este artículo se describe format_timespan () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: ba4dffa50c605e9346807f28222809af7637ff09
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227288"
---
# <a name="format_timespan"></a>format_timespan()

Da formato a un intervalo de tiempo según el formato proporcionado.

```kusto
format_timespan(time(14.02:03:04.12345), 'h:m:s.fffffff') == "2:3:4.1234500"
```

**Sintaxis**

`format_timespan(`*intervalo* `,` de tiempo *formato*`)`

**Argumentos**

* `timespan`: valor de un tipo `timespan` .
* `format`: especificador de formato cadena, que consta de uno o varios [elementos de formato](#supported-formats).

**Devuelve**

Cadena con el resultado de formato.

## <a name="supported-formats"></a>Formatos compatibles

|Especificador de formato   |Descripción    |Ejemplos
|---|---|---
|`d`-`dddddddd` |Número de días completos de un intervalo de tiempo. Se rellena con ceros si es necesario.|   15.13:45:30: d-> 15, DD-> 15, DDD-> 015
|`f`    |Las décimas de segundo del intervalo de tiempo. |15.13:45:30.6170000-> 6, 15.13:45:30.05-> 0
|`ff`   |Centésimas de segundo del intervalo de tiempo. |15.13:45:30.6170000-> 61, 15.13:45:30.0050000-> 00
|`fff`  |Los milisegundos del intervalo de tiempo. |6/15/2009 13:45:30.617-> 617, 6/15/2009 13:45:30.0005-> 000
|`ffff` |Las diezmilésimas de segundo de un intervalo de tiempo. |15.13:45:30.6175000-> 6175, 15.13:45:30.0000500-> 0000
|`fffff`    |Las cien milésimas de segundo del intervalo de tiempo. |15.13:45:30.6175400-> 61754, 15.13:45:30.000005-> 00000
|`ffffff`   |Las millonésimas de segundo de un intervalo de tiempo. |15.13:45:30.6175420-> 617542, 15.13:45:30.0000050-> 000000
|`fffffff`  |Las diez millonésimas de segundo del intervalo de tiempo. |15.13:45:30.6175425-> 6175425, 15.13:45:30.0001150-> 0001150
|`F`    |Si es distinto de cero, las décimas de segundo del intervalo de tiempo. |15.13:45:30.6170000-> 6, 15.13:45:30.0500000-> (sin salida)
|`FF`   |Si es distinto de cero, las centésimas de segundo del intervalo de tiempo. |15.13:45:30.6170000-> 61, 15.13:45:30.0050000-> (sin salida)
|`FFF`  |Si es distinto de cero, los milisegundos del intervalo de tiempo. |15.13:45:30.6170000-> 617, 15.13:45:30.0005000-> (sin salida)
|`FFFF` |Si es distinto de cero, las diezmilésimas de segundo de un intervalo de tiempo. |15.13:45:30.5275000-> 5275, 15.13:45:30.0000500-> (sin salida)
|`FFFFF`    |Si es distinto de cero, las cien milésimas de segundo del intervalo de tiempo. |15.13:45:30.6175400-> 61754, 15.13:45:30.0000050-> (sin salida)
|`FFFFFF`   |Si es distinto de cero, las millonésimas de segundo del intervalo de tiempo. |15.13:45:30.6175420-> 617542, 15.13:45:30.0000050-> (sin salida)
|`FFFFFFF`  |Si es distinto de cero, las diez millonésimas de segundo del intervalo de tiempo. |15.13:45:30.6175425-> 6175425, 15.13:45:30.0001150-> 000115
|`h`    |Número de horas completas de un intervalo de tiempo que no se cuentan como parte de los días. Las horas con un solo dígito no se escriben con un cero a la izquierda. |15.01:45:30-> 1, 15.13:45:30-> 1
|`hh`   |Número de horas completas de un intervalo de tiempo que no se cuentan como parte de los días. Las horas con un solo dígito se escriben con un cero a la izquierda. |15.01:45:30-> 01, 15.13:45:30-> 01
|`H`    |La hora, usando un reloj de 24 horas de 0 a 23. |15.01:45:30-> 1, 15.13:45:30-> 13
|`HH`   |La hora, usando un reloj de 24 horas de 00 a 23. |15.01:45:30-> 01, 15.13:45:30-> 13
|`m`    |Número de minutos completos de un intervalo de tiempo que no se incluyen como parte de las horas o los días. Los minutos con un solo dígito no se escriben con un cero a la izquierda. |15.01:09:30-> 9, 15.13:29:30-> 29
|`mm`   |Número de minutos completos de un intervalo de tiempo que no se incluyen como parte de las horas o los días. Los minutos con un solo dígito se escriben con un cero a la izquierda. |15.01:09:30-> 09, 15.01:45:30-> 45
|`s`    |Número de segundos completos de un intervalo de tiempo que no se incluyen como parte de las horas, los días o los minutos. Los segundos con un solo dígito no se escriben con un cero a la izquierda. |15.13:45:09-> 9
|`ss`   |Número de segundos completos de un intervalo de tiempo que no se incluyen como parte de las horas, los días o los minutos. Los segundos con un solo dígito se escriben con un cero a la izquierda. |15.13:45:09-> 09

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
let t = time(29.09:00:05.12345);
print 
v1=format_timespan(t, 'dd.hh:mm:ss:FF'),
v2=format_timespan(t, 'ddd.h:mm:ss [fffffff]')
```

|v1|v2|
|---|---|
|29.09:00:05:12|029.9:00:05 [1234500]|
