---
title: url_encode_component() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe url_encode_component() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/17/2020
ms.openlocfilehash: bfdb40f362aa680a68bd8871769eecb5a2da19e6
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505073"
---
# <a name="url_encode_component"></a>url_encode_component()

La función convierte los caracteres de la URL de entrada en un formato que se puede transmitir a través de Internet. 

Puede encontrar información detallada sobre la codificación y decodificación de URL [aquí](https://en.wikipedia.org/wiki/Percent-encoding).
Se diferencia de [url_encode](./urlencodefunction.md) por codificar espacios como '20%' y no como '+'.

**Sintaxis**

`url_encode_component(`*Url*`)`

**Argumentos**

* *url*: URL de entrada (cadena).  

**Devuelve**

URL (cadena) convertida en un formato que se puede transmitir a través de Internet.

**Ejemplos**

```kusto
let url = @'https://www.bing.com/hello word/';
print original = url, encoded = url_encode_component(url)
```

|original|Codificado|
|---|---|
|https://www.bing.com/hellopalabra/|https%3a%2f%2fwww.bing.com%2fhello%20word|


 