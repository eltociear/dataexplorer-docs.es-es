---
title: format_timespan() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe format_timespan() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 2cc894d3b10cd3e4badd1ff7b498c23618c76818
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515001"
---
# <a name="format_timespan"></a>format_timespan()

Da formato a un intervalo de tiempo según el formato proporcionado.

```kusto
format_timespan(time(14.02:03:04.12345), 'h:m:s.fffffff') == "2:3:4.1234500"
```

**Sintaxis**

`format_timespan(`formato *de intervalo* `,` *de* tiempo`)`

**Argumentos**

* `timespan`: valor de `timespan`un tipo .
* `format`: cadena de especificador de formato, que consta de uno o más elementos de [formato.](#supported-formats)

**Devuelve**

Cadena con el resultado del formato.

## <a name="supported-formats"></a>Formatos compatibles

|Especificador de formato   |Descripción    |Ejemplos
|---|---|---
|`d`-`dddddddd` |Número de días completos de un intervalo de tiempo. Acolchado con ceros si es necesario.|   15.13:45:30: d -> 15, dd -> 15, ddd -> 015
|`f`    |Las décimas de segundo en el intervalo de tiempo. |15.13:45:30.6170000 -> 6, 15.13:45:30.05 -> 0
|`ff`   |Las centésimas de segundo en el intervalo de tiempo. |15.13:45:30.6170000 -> 61, 15.13:45:30.0050000 -> 00
|`fff`  |Los milisegundos en el intervalo de tiempo. |6/15/2009 13:45:30.617 -> 617, 6/15/2009 13:45:30.0005 -> 000
|`ffff` |Las diez milésimas de segundo en el intervalo de tiempo. |15.13:45:30.6175000 -> 6175, 15.13:45:30.0000500 -> 0000
|`fffff`    |Las cien milésimas de segundo en el intervalo de tiempo. |15.13:45:30.6175400 -> 61754, 15.13:45:30.000005 -> 00000
|`ffffff`   |Las millonésimas de segundo en el intervalo de tiempo. |15.13:45:30.6175420 -> 617542, 15.13:45:30.0000005 -> 000000
|`fffffff`  |Las diez millonésimas de segundo en el intervalo de tiempo. |15.13:45:30.6175425 -> 6175425, 15.13:45:30.0001150 -> 0001150
|`F`    |Si no es cero, las décimas de segundo en el intervalo de tiempo. |15.13:45:30.6170000 -> 6, 15.13:45:30.0500000 -> (sin salida)
|`FF`   |Si no es cero, las centésimas de segundo en el intervalo de tiempo. |15.13:45:30.6170000 -> 61, 15.13:45:30.0050000 -> (sin salida)
|`FFF`  |Si no es cero, los milisegundos en el intervalo de tiempo. |15.13:45:30.6170000 -> 617, 15.13:45:30.0005000 -> (sin salida)
|`FFFF` |Si no es cero, las diez milésimas de segundo en el intervalo de tiempo. |15.13:45:30.5275000 -> 5275, 15.13:45:30.0000500 -> (sin salida)
|`FFFFF`    |Si no es cero, las cien milésimas de segundo en el intervalo de tiempo. |15.13:45:30.6175400 -> 61754, 15.13:45:30.0000050 -> (sin salida)
|`FFFFFF`   |Si no es cero, las millonésimas de segundo en el intervalo de tiempo. |15.13:45:30.6175420 -> 617542, 15.13:45:30.0000005 -> (sin salida)
|`FFFFFFF`  |Si no es cero, las diez millonésimas de segundo en el intervalo de tiempo. |15.13:45:30.6175425 -> 6175425, 15.13:45:30.0001150 -> 000115
|`h`    |Número de horas completas de un intervalo de tiempo que no se cuentan como parte de los días. Las horas con un solo dígito no se escriben con un cero a la izquierda. |15.01:45:30 -> 1, 15.13:45:30 -> 1
|`hh`   |Número de horas completas de un intervalo de tiempo que no se cuentan como parte de los días. Las horas con un solo dígito se escriben con un cero a la izquierda. |15.01:45:30 -> 01, 15.13:45:30 -> 01
|`H`    |La hora, usando un reloj de 24 horas de 0 a 23. |15.01:45:30 -> 1, 15.13:45:30 -> 13
|`HH`   |La hora, usando un reloj de 24 horas de 00 a 23. |15.01:45:30 -> 01, 15.13:45:30 -> 13
|`m`    |Número de minutos completos de un intervalo de tiempo que no se incluyen como parte de las horas o los días. Los minutos con un solo dígito no se escriben con un cero a la izquierda. |15.01:09:30 -> 9, 15.13:29:30 -> 29
|`mm`   |Número de minutos completos de un intervalo de tiempo que no se incluyen como parte de las horas o los días. Los minutos con un solo dígito se escriben con un cero a la izquierda. |15.01:09:30 -> 09, 15.01:45:30 -> 45
|`s`    |Número de segundos completos de un intervalo de tiempo que no se incluyen como parte de las horas, los días o los minutos. Los segundos con un solo dígito no se escriben con un cero a la izquierda. |15.13:45:09 -> 9
|`ss`   |Número de segundos completos de un intervalo de tiempo que no se incluyen como parte de las horas, los días o los minutos. Los segundos con un solo dígito se escriben con un cero a la izquierda. |15.13:45:09 -> 09

**Delimeters soportados**

El especificador de formato puede incluir los siguientes caracteres de delirios:

|Delímetro|Comentario|
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

```kusto
let t = time(29.09:00:05.12345);
print 
v1=format_timespan(t, 'dd.hh:mm:ss:FF'),
v2=format_timespan(t, 'ddd.h:mm:ss [fffffff]')
```

|v1|v2|
|---|---|
|29.09:00:05:12|029.9:00:05 [1234500]|
