---
title: reverse() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe reverse() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e95246e0586dff7dd89dc2658c7fae08b1bbaddf
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510309"
---
# <a name="reverse"></a>reverse()

La función hace que se revierta la cadena de entrada.

Si el valor de entrada no es de tipo cadena, la función convierte forzosamente el valor en string.

**Sintaxis**

`reverse(`*Fuente*`)`

**Argumentos**

* *fuente*: valor de entrada.  

**Devuelve**

El orden inverso de un valor de cadena.

**Ejemplos**

```kusto
print str = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
| extend rstr = reverse(str)
```

|str|rstr|
|---|---|
|ABCDEFGHIJKLMNOPQRSTUVWXYZ|ZYXWVUTSRQPONMLKJIHGFEDCBA|


```kusto
print ['int'] = 12345, ['double'] = 123.45, 
['datetime'] = datetime(2017-10-15 12:00), ['timespan'] = 3h
| project rint = reverse(['int']), rdouble = reverse(['double']), 
rdatetime = reverse(['datetime']), rtimespan = reverse(['timespan'])
```

|rint|rdouble|rdatetime|rtimespan|
|---|---|---|---|
|54321|54.321|Z0000000.00:00:21T51-01-7102|00:00:30|




 