---
title: url_encode() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe url_encode() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/17/2020
ms.openlocfilehash: 913be2d20af413f8ba89192f4db57e60fc6d7b27
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505056"
---
# <a name="url_encode"></a>url_encode()

La función convierte los caracteres de la URL de entrada en un formato que se puede transmitir a través de Internet. 

Puede encontrar información detallada sobre la codificación y decodificación de URL [aquí](https://en.wikipedia.org/wiki/Percent-encoding).
Se diferencia de [url_encode_component](./urlencodecomponentfunction.md) por la codificación de espacios como '+' y no como '20%' (consulte application/x-www-form-urlencoded [aquí](https://en.wikipedia.org/wiki/Percent-encoding)).

**Sintaxis**

`url_encode(`*Url*`)`

**Argumentos**

* *url*: URL de entrada (cadena).  

**Devuelve**

URL (cadena) convertida en un formato que se puede transmitir a través de Internet.

**Ejemplos**

```kusto
let url = @'https://www.bing.com/hello word';
print original = url, encoded = url_encode(url)
```

|original|Codificado|
|---|---|
|https://www.bing.com/hellopalabra/|https%3a%2f%2fwww.bing.com%2fhello+word|


 