---
title: 'base64_decode_toarray (): Explorador de datos de Azure'
description: En este artículo se describe base64_decode_toarray () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/22/2019
ms.openlocfilehash: df10ea2bbdf4e48d32c087d35eb9361fc4d697b8
ms.sourcegitcommit: ae72164adc1dc8d91ef326e757376a96ee1b588d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2020
ms.locfileid: "84717213"
---
# <a name="base64_decode_toarray"></a>base64_decode_toarray()

Descodifica una cadena Base64 en una matriz de valores Long.

**Sintaxis**

`base64_decode_toarray(`*String@*`)`

**Argumentos**

* *Cadena*: cadena de entrada que se va a descodificar de la cadena de Base64 a UTF8.

**Devuelve**

Devuelve una matriz de valores Long descodificados de una cadena Base64.

* Para descodificar cadenas Base64 en una cadena UTF-8, vea [base64_decode_tostring ()](base64_decode_tostringfunction.md)
* Para codificar cadenas en una cadena Base64, vea [base64_encode_tostring ()](base64_encode_tostringfunction.md)

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print Quine=base64_decode_toarray("S3VzdG8=")  
// 'K', 'u', 's', 't', 'o'
```

|Quine|
|-----|
|[75.117.115.116.111]|

Si intenta descodificar una cadena Base64 que se generó a partir de una codificación UTF-8 no válida, se devolverá "null":

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print Empty=base64_decode_toarray("U3RyaW5n0KHR0tGA0L7Rh9C60LA=")
```

|Empty|
|-----|
||
