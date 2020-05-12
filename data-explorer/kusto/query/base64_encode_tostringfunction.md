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
ms.openlocfilehash: 332ff6bedd268dd79be020ff1dc4d0591ed486f7
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225316"
---
# <a name="base64_encode_tostring"></a>base64_encode_tostring()

Codifica una cadena como una cadena Base64.

**Sintaxis**

`base64_encode_tostring(`*String@*`)`

**Argumentos**

* *Cadena*: cadena de entrada que se va a codificar como cadena Base64.

**Devuelve**

Devuelve la cadena codificada como cadena Base64.

* Para descodificar cadenas Base64 en una cadena UTF-8, vea [base64_decode_tostring ()](base64_decode_tostringfunction.md)
* Para descodificar cadenas Base64 en una matriz de valores Long [, vea base64_decode_toarray ()](base64_decode_toarrayfunction.md)


**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print Quine=base64_encode_tostring("Kusto")
```

|Quine   |
|--------|
|S3VzdG8 =|
