---
title: trim() - Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe trim() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: aef2a61e5ac13fe9af9d8bc0dd130f3d085a3604
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505600"
---
# <a name="trim"></a>trim()

Quita todas las coincidencias iniciales y finales de la expresión regular especificada.

**Sintaxis**

`trim(`*regex*`,` *texto* regex`)`

**Argumentos**

* *regex*: Cadena o [expresión regular](re2.md) que se va a recortar desde el principio y/o el final del *texto*.  
* *texto*: Una cadena.

**Devuelve**

*texto* después de recortar las coincidencias de *regex* que se encuentran al principio y/o al final del *texto.*

**Ejemplo**

La instrucción a continuación recorta la *subcadena* desde el inicio y el final del *string_to_trim:*

```kusto
let string_to_trim = @"--https://bing.com--";
let substring = "--";
print string_to_trim = string_to_trim, trimmed_string = trim(substring,string_to_trim)
```

|string_to_trim|trimmed_string|
|---|---|
|--https://bing.com--|https://bing.com|

La siguiente instrucción recorta todos los caracteres que no son de palabra desde el principio y el final de la cadena:

```kusto
range x from 1 to 5 step 1
| project str = strcat("-  ","Te st",x,@"// $")
| extend trimmed_str = trim(@"[^\w]+",str)
```

|str|trimmed_str|
|---|---|
|- Te st1// $|Te st1|
|- Te st2// $|Te st2|
|- Te st3// $|Te st3|
|- Te st4// $|Te st4|
|- Te st5// $|Te st5|


 