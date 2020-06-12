---
title: 'base64_encode_tostring (): Explorador de datos de Azure'
description: En este artículo se describe base64_encode_tostring () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/22/2019
ms.openlocfilehash: c414f1bdb83850bc6ec6065314bc7c8662ab0ed2
ms.sourcegitcommit: ae72164adc1dc8d91ef326e757376a96ee1b588d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2020
ms.locfileid: "84717094"
---
# <a name="base64_encode_tostring"></a>base64_encode_tostring()

Codifica una cadena como una cadena Base64.

**Sintaxis**

`base64_encode_tostring(`*String@*`)`

**Argumentos**

* *Cadena*: cadena de entrada que se va a codificar como cadena Base64.

**Devuelve**

Devuelve la cadena codificada como cadena Base64.

* Para descodificar cadenas Base64 en cadenas UTF-8, vea [base64_decode_tostring ()](base64_decode_tostringfunction.md)
* Para descodificar cadenas Base64 en una matriz de valores Long, vea [base64_decode_toarray ()](base64_decode_toarrayfunction.md)


**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print Quine=base64_encode_tostring("Kusto")
```

|Quine   |
|--------|
|S3VzdG8 =|

