---
title: base64_encode_fromarray() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe base64_encode_fromarray() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/11/2020
ms.openlocfilehash: f601463fd6751be7064892e70e5b2235f96a20ff
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518078"
---
# <a name="base64_encode_fromarray"></a>base64_encode_fromarray()

Codifica una cadena base64 a partir de una matriz de bytes.

**Sintaxis**

`base64_encode_fromarray(`*BytesArray*`)`

**Argumentos**

* *BytesArray*: Matriz de bytes de entrada que se codificará en cadena base64.

**Devuelve**

Devuelve la cadena base64 codificada desde la matriz bytes.

* Para decodificar cadenas base64 en una cadena UTF-8, consulte [base64_decode_tostring()](base64_decode_tostringfunction.md)
* Para codificar cadenas en la cadena base64, consulte [base64_encode_tostring()](base64_encode_tostringfunction.md)
* Esta función es la inversa de [base64_decode_toarray()](base64_decode_toarrayfunction.md)

**Ejemplo**

```kusto
let bytes_array = toscalar(print base64_decode_toarray("S3VzdG8="));
print decoded_base64_string = base64_encode_fromarray(bytes_array)
```

|decoded_base64_string|
|---|
|S3VzdG8|


Al intentar codificar una cadena base64 a partir de una matriz de bytes no válida que se generó a partir de una cadena codificada UTF-8 no válida, se devolverá null:

```kusto
let empty_bytes_array = toscalar(print base64_decode_toarray("U3RyaW5n0KHR0tGA0L7Rh9C60LA"));
print empty_string = base64_encode_fromarray(empty_bytes_array)
```

|empty_string|
|---|
||