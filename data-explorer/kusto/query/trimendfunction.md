---
title: trim_end() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe trim_end() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a6f6ffc264cb436fc61d74f08dfded915caa05d4
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505651"
---
# <a name="trim_end"></a>trim_end()

Quita la coincidencia final de la expresión regular especificada.

**Sintaxis**

`trim_end(`*regex*`,` *texto* regex`)`

**Argumentos**

* *regex*: Cadena o [expresión regular](re2.md) que se va a recortar desde el final del *texto*.  
* *texto*: Una cadena.

**Devuelve**

*texto* después de recortar coincidencias de *regex* encontrado en el final del *texto.*

**Ejemplo**

La instrucción a continuación recorta la *subcadena* desde el final de *string_to_trim:*

```kusto
let string_to_trim = @"bing.com";
let substring = ".com";
print string_to_trim = string_to_trim,trimmed_string = trim_end(substring,string_to_trim)
```

|string_to_trim|trimmed_string|
|--------------|--------------|
|bing.com      |bing          |

La siguiente instrucción recorta todos los caracteres que no son de palabra desde el final de la cadena:

```kusto
print str = strcat("-  ","Te st",x,@"// $")
| extend trimmed_str = trim_end(@"[^\w]+",str)
```

|str          |trimmed_str|
|-------------|-----------|
|- Te st1// $|- Te st1  |
|- Te st2// $|- Te st2  |
|- Te st3// $|- Te st3  |
|- Te st4// $|- Te st4  |
|- Te st5// $|- Te st5  |