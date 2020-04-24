---
title: trim_start() - Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe trim_start() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1d4ae71f73e76005f89766d974192c8eb24cd74d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505583"
---
# <a name="trim_start"></a>trim_start()

Quita la coincidencia inicial de la expresión regular especificada.

**Sintaxis**

`trim_start(`*regex*`,` *texto* regex`)`

**Argumentos**

* *regex*: Cadena o [expresión regular](re2.md) que se va a recortar desde el principio del *texto*.  
* *texto*: Una cadena.

**Devuelve**

*texto* después de recortar la coincidencia de *regex* encontrado en el principio del *texto.*

**Ejemplo**

La instrucción a continuación recorta la *subcadena* desde el inicio de *string_to_trim:*

```kusto
let string_to_trim = @"https://bing.com";
let substring = "https://";
print string_to_trim = string_to_trim,trimmed_string = trim_start(substring,string_to_trim)
```

|string_to_trim|trimmed_string|
|---|---|
|https://bing.com|bing.com|

La siguiente instrucción recorta todos los caracteres que no son de palabra desde el principio de la cadena:

```kusto
range x from 1 to 5 step 1
| project str = strcat("-  ","Te st",x,@"// $")
| extend trimmed_str = trim_start(@"[^\w]+",str)
```

|str|trimmed_str|
|---|---|
|- Te st1// $|Te st1// $|
|- Te st2// $|Te st2// $|
|- Te st3// $|Te st3// $|
|- Te st4// $|Te st4// $|
|- Te st5// $|Te st5// $|

 