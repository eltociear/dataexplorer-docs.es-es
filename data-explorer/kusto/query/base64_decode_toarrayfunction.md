---
title: base64_decode_toarray() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe base64_decode_toarray() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/22/2019
ms.openlocfilehash: 80a702f112fd4d7b88b4011b3f615cf34acff62c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518197"
---
# <a name="base64_decode_toarray"></a>base64_decode_toarray()

Descodifica una cadena base64 en una matriz de valores largos.

**Sintaxis**

`base64_decode_toarray(`*Cadena*`)`

**Argumentos**

* *String*: Cadena de entrada que se va a decodificar de la cadena base64 a UTF8-8.

**Devuelve**

Devuelve la matriz de valores largos ecoded de la cadena base64.

* Para decodificar cadenas base64 en una cadena UTF-8, consulte [base64_decode_tostring()](base64_decode_tostringfunction.md)
* Para codificar cadenas en la cadena base64, consulte [base64_encode_tostring()](base64_encode_tostringfunction.md)

**Ejemplo**

```kusto
print Quine=base64_decode_toarray("S3VzdG8=")  
// 'K', 'u', 's', 't', 'o'
```

|Quine|
|-----|
|[75,117,115,116,111]|

Al intentar decodificar una cadena base64 que se generó a partir de una codificación UTF-8 no válida, se devolverá null:

```kusto
print Empty=base64_decode_toarray("U3RyaW5n0KHR0tGA0L7Rh9C60LA=")
```

|Vacío|
|-----|
||