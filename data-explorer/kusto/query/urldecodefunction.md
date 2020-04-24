---
title: url_decode() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe url_decode() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 3ffa5052a2fc30387be118683ec1df6f34f7346f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505158"
---
# <a name="url_decode"></a>url_decode()

La función convierte la dirección URL codificada en una representación de URL a normal. 

Puede encontrar información detallada sobre la decodificación y codificación de URL [aquí](https://en.wikipedia.org/wiki/Percent-encoding).

**Sintaxis**

`url_decode(`*url codificada*`)`

**Argumentos**

* *URL codificada:* URL codificada (cadena).  

**Devuelve**

DIRECCIÓN URL (cadena) en una representación regular.

**Ejemplos**

```kusto
let url = @'https%3a%2f%2fwww.bing.com%2f';
print original = url, decoded = url_decode(url)
```

|original|Decodificado|
|---|---|
|https%3a%2f%2fwww.bing.com%2f|https://www.bing.com/|



 