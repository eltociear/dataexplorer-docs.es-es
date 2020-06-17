---
title: 'base64_decode_tostring (): Explorador de datos de Azure'
description: En este artículo se describe base64_decode_tostring () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/22/2019
ms.openlocfilehash: ffef2fd609075b0d5e5af5c4064e079c27cd8c94
ms.sourcegitcommit: 3848b8db4c3a16bda91c4a5b7b8b2e1088458a3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2020
ms.locfileid: "84818498"
---
# <a name="base64_decode_tostring"></a>base64_decode_tostring()

Descodifica una cadena Base64 en una cadena UTF-8.

**Sintaxis**

`base64_decode_tostring(`*String@*`)`

**Argumentos**

* *Cadena*: cadena de entrada que se va a descodificar de Base64 a UTF8-8 cadena.

**Devuelve**

Devuelve la cadena UTF-8 descodificada de la cadena Base64.

* Para descodificar cadenas Base64 en una matriz de valores Long, vea [base64_decode_toarray ()](base64_decode_toarrayfunction.md)
* Para descodificar cadenas en una cadena Base64, vea [base64_encode_tostring ()](base64_encode_tostringfunction.md)

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print Quine=base64_decode_tostring("S3VzdG8=")
```

|Quine|
|-----|
|Kusto|

Al intentar descodificar una cadena Base64 que se generó a partir de una codificación UTF-8 no válida, se devolverá null:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print Empty=base64_decode_tostring("U3RyaW5n0KHR0tGA0L7Rh9C60LA=")
```

|Empty|
|-----|
||
