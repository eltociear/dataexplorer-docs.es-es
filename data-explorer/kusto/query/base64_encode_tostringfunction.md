---
title: base64_encode_tostring() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe base64_encode_tostring() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 06/22/2019
ms.openlocfilehash: a80b0aa0e3f7e5f330da87f93bbad44e587bcdaf
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518061"
---
# <a name="base64_encode_tostring"></a>base64_encode_tostring()

Codifica una cadena como cadena base64

**Sintaxis**

`base64_encode_tostring(`*Cadena*`)`

**Argumentos**

* *String*: Cadena de entrada que se codificará como cadena base64.

**Devuelve**

Devuelve la cadena codificada como cadena base64.

* Para decodificar cadenas base64 en una cadena UTF-8, consulte [base64_decode_tostring()](base64_decode_tostringfunction.md)
* Para decodificar cadenas base64 a una matriz de valores largos, consulte [base64_decode_toarray()](base64_decode_toarrayfunction.md)


**Ejemplo**

```kusto
print Quine=base64_encode_tostring("Kusto")
```

|Quine   |
|--------|
|S3VzdG8|
