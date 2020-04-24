---
title: strcmp() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe strcmp() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 62ebcbfa71ebf8a29f3a8a1559feb91f96e0a2bf
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506909"
---
# <a name="strcmp"></a>strcmp()

Compara dos cadenas.

La función comienza a comparar el primer carácter de cada cadena. Si son iguales entre sí, continúa con los siguientes pares hasta que los caracteres difieren o hasta que se alcanza el final de una cadena más corta.

**Sintaxis**

`strcmp(`*string1* `,` *string2*`)` 

**Argumentos**

* *string1*: primera cadena de entrada para la comparación. 
* *string2*: segunda cadena de entrada para la comparación.

**Devuelve**

Devuelve un valor entero que indica la relación entre las cadenas:
* *<0* - el primer carácter que no coincide tiene un valor menor en string1 que en string2
* *0* - el contenido de ambas cadenas es igual
* *>0* - el primer carácter que no coincide tiene un valor mayor en string1 que en string2

**Ejemplos**

```
datatable(string1:string, string2:string)
["ABC","ABC",
"abc","ABC",
"ABC","abc",
"abcde","abc"]
| extend result = strcmp(string1,string2)
```

|string1|cadena2|resultado|
|---|---|---|
|ABC|ABC|0|
|abc|ABC|1|
|ABC|abc|-1|
|Abcde|abc|1|