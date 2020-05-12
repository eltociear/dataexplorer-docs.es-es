---
title: 'base64_encode_fromarray (): Explorador de datos de Azure'
description: En este artículo se describe base64_encode_fromarray () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/11/2020
ms.openlocfilehash: bee6471ef2cf2a2cd484af8ce84d70cce749d5e0
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225333"
---
# <a name="base64_encode_fromarray"></a>base64_encode_fromarray()

Codifica una cadena Base64 a partir de una matriz de bytes.

**Sintaxis**

`base64_encode_fromarray(`*BytesArray*`)`

**Argumentos**

* *BytesArray*: matriz de bytes de entrada que se va a codificar en una cadena Base64.

**Devuelve**

Devuelve la cadena Base64 codificada a partir de la matriz de bytes.

* Para descodificar cadenas Base64 en una cadena UTF-8, vea [base64_decode_tostring ()](base64_decode_tostringfunction.md)
* Para codificar cadenas en una cadena Base64 [, vea base64_encode_tostring ()](base64_encode_tostringfunction.md)
* Esta función es la inversa de [base64_decode_toarray ()](base64_decode_toarrayfunction.md)

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
let bytes_array = toscalar(print base64_decode_toarray("S3VzdG8="));
print decoded_base64_string = base64_encode_fromarray(bytes_array)
```

|decoded_base64_string|
|---|
|S3VzdG8 =|


Al intentar codificar una cadena Base64 a partir de una matriz de bytes no válida que se generó a partir de una cadena con codificación UTF-8 no válida, se devolverá null:

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
let empty_bytes_array = toscalar(print base64_decode_toarray("U3RyaW5n0KHR0tGA0L7Rh9C60LA"));
print empty_string = base64_encode_fromarray(empty_bytes_array)
```

|empty_string|
|---|
||
