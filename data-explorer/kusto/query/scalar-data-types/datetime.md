---
title: 'El tipo de datos datetime: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el tipo de datos datetime en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: a5a234c49a5bb5b04ccde49e7fb3702d927e38b5
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509918"
---
# <a name="the-datetime-data-type"></a>El tipo de datos datetime

El `datetime` `date`tipo de datos ( ) representa un instante en el tiempo, normalmente expresado como una fecha y hora del día.
Los valores van desde 00:00:00 (medianoche), 1 de enero de 0001 Anno Domini (Era Común) hasta las 11:59:59 p.m., 31 de diciembre de 9999 d.C. (C.E.) en el calendario gregoriano. 

Los valores de tiempo se miden en unidades de 100 nanosegundos llamadas ticks, y una fecha determinada es el número de ticks desde las 12:00 de la medianoche, el 1 de enero de 0001 d.C. (C.E.) en el calendario GregorianCalendar (excluyendo las marcas que se agregarían por segundos bisiestos).
Por ejemplo, un valor de ticks de 31241376000000000 representa la fecha, viernes, 01 de enero, 0100 12:00:00 medianoche.
Esto a veces se llama "un momento en el tiempo lineal".

> [!WARNING]
> Un `datetime` valor en Kusto siempre está en la zona horaria UTC. Mostrar `datetime` valores en otras zonas horarias es responsabilidad de la aplicación de usuario que muestra los datos, no una propiedad de los datos en sí. Si se requiere que los valores de zona horaria se mantengan como parte de los datos, se debe utilizar una columna independiente (proporcionando información de desplazamiento relativa a UTC).

## <a name="datetime-literals"></a>datetime (literales)

Los literales `datetime` de `datetime(`tipo tienen el *valor*`)`de sintaxis, donde se admiten varios formatos para *value*, como se indica en la tabla siguiente:

|Ejemplo                                                     |Valor                                                         |
|------------------------------------------------------------|--------------------------------------------------------------|
|`datetime(2015-12-31 23:59:59.9)`<br/>`datetime(2015-12-31)`|Las horas se expresan siempre en formato UTC. La omisión de la fecha genera una hora del día actual.|
|`datetime(null)`                                            |Consulte [valores nulos](null-values.md).                            |
|`now()`                                                     |La hora actual.                                             |
|`now(`-*Timespan*`)`                                        |`now()-`*Timespan*                                            |
|`ago(`*Timespan*`)`                                         |`now()-`*Timespan*                                            |

`now()`e `ago()` indicar `datetime` un valor en comparación con el momento en que Kusto comenzó a ejecutar la consulta. Estos pueden aparecer varias veces en la misma consulta y se usará un único valor para todos ellos.
(En otras palabras, `now(-x) - ago(x)` expresiones `timespan` como siempre se evalúan como un valor de cero.)

## <a name="supported-formats"></a>Formatos compatibles

Hay varios formatos para `datetime` los que se admiten como [literales datetime()](#datetime-literals) y la función [todatetime().](../todatetimefunction.md)

> [!WARNING]
> Se **recomienda encarecidamente** utilizar sólo los formatos ISO 8601.

### <a name="iso-8601"></a>[ISO 8601](https://www.iso.org/iso/home/standards/iso8601.htm)

|Formato|Ejemplo|
|------|-------|
|%Y-%m-%dT%H:%M:%s%z|2014-05-25T08:20:03.123456Z|
|%Y-%m-%dT%H:%M:%s"|2014-05-25T08:20:03.123456|
|%Y-%m-%dT%H:%M"|2014-05-25T08:20|
|%Y-%m-%d %H:%M:%s%z"|2014-11-08 15:55:55.123456Z|
|%Y-%m-%d %H:%M:%s"|2014-11-08 15:55:55|
|%Y-%m-%d %H:%M"|2014-11-08 15:55|
|%Y-%m-%d|2014-11-08|

### <a name="rfc-822"></a>[RFC 822](https://www.ietf.org/rfc/rfc0822.txt)

|Formato|Ejemplo|
|------|-------|
|%w, %e %b %r %H:%M:%s %Z|Sáb, 8 Nov 14 15:05:02 GMT|
|%w, %e %b %r %H:%M:%s|Sáb, 8 Nov 14 15:05:02|
|%w, %e %b %r %H:%M|Sáb, 8 Nov 14 15:05|
|%w, %e %b %r %H:%M %Z|Sáb, 8 Nov 14 15:05 GMT|
|%e %b %r %H:%M:%s %Z|8 Nov 14 15:05:02 GMT|
|%e %b %r %H:%M:%s"|8 Nov 14 15:05:02|
|%e %b %r %H:%M"|8 Nov 14 15:05|
|%e %b %r %H:%M %Z"|8 Nov 14 15:05 GMT|

### <a name="rfc-850"></a>[RFC 850](https://tools.ietf.org/html/rfc850)

|Formato|Ejemplo|
|------|-------|
|%w, %e-%b-%r %H:%M:%s %Z|Sábado, 08-Nov-14 15:05:02 GMT|
|%w, %e-%b-%r %H:%M:%s|Sábado, 08-Nov-14 15:05:02|
|%w, %e-%b-%r %H:%M %Z|Sábado, 08-Nov-14 15:05 GMT|
|%w, %e-%b-%r %H:%M|Sábado, 08-Nov-14 15:05|
|%e-%b-%r %H:%M:%s %Z|08-Nov-14 15:05:02 GMT|
|%e-%b-%r %H:%M:%s"|08-Nov-14 15:05:02|
|%e-%b-%r %H:%M %Z"|08-Nov-14 15:05 GMT|
|%e-%b-%r %H:%M"|08-Nov-14 15:05|


### <a name="sortable"></a>Sortable 

|Formato|Ejemplo|
|------|-------|        
|%Y-%n-%e %H:%M:%s|2014-11-08 15:05:25|
|%Y-%n-%e %H:%M:%s %Z|2014-11-08 15:05:25 GMT|
|%Y-%n-%e %H:%M|2014-11-08 15:05|
|%Y-%n-%e %H:%M %Z|2014-11-08 15:05 GMT|
|%Y-%n-%eT%H:%M:%s|2014-11-08T15:05:25|
|%Y-%n-%eT%H:%M:%s %Z|2014-11-08T15:05:25 GMT|
|%Y-%n-%eT%H:%M|2014-11-08T15:05|
|%Y-%n-%eT%H:%M %Z"|2014-11-08T15:05 GMT|