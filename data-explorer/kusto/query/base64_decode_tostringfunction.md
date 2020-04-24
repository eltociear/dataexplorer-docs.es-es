---
title: base64_decode_tostring() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe base64_decode_tostring() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/22/2019
ms.openlocfilehash: a821fb07d62d5bca3982cc4c48b8e0457641f3f2
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518180"
---
# <a name="base64_decode_tostring"></a>base64_decode_tostring()

Descodifica una cadena base64 en una cadena UTF-8

**Sintaxis**

`base64_decode_tostring(`*Cadena*`)`

**Argumentos**

* *String*: Cadena de entrada que se va a decodificar de la cadena base64 a UTF8-8.

**Devuelve**

Devuelve la cadena UTF-8 descodificada de la cadena base64.

* Para decodificar cadenas base64 a una matriz de valores largos, consulte [base64_decode_toarray()](base64_decode_toarrayfunction.md)
* Para codificar cadenas en la cadena base64, consulte [base64_encode_tostring()](base64_encode_tostringfunction.md)

**Ejemplo**

```kusto
print Quine=base64_decode_tostring("S3VzdG8=")
```

|Quine|
|-----|
|Kusto|

Al intentar decodificar una cadena base64 que se generó a partir de una codificación UTF-8 no válida, se devolverá null:

```kusto
print Empty=base64_decode_tostring("U3RyaW5n0KHR0tGA0L7Rh9C60LA=")
```

|Vacío|
|-----|
||