---
title: strrep() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe strrep() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 39b398e8fadb400c25cfeb97487c2ecf0669ad83
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506722"
---
# <a name="strrep"></a>strrep()

Repite la cantidad de veces proporcionada por la [cadena.](./scalar-data-types/string.md)

* En caso de que el primer o tercer argumento no sea de un tipo de cadena, se convertirá forzosamente en cadena.

**Sintaxis**

`strrep(`*value*,*multiplier*,[*delimitador*]`)`

**Argumentos**

* *valor*: expresión de entrada
* *multiplicador*: valor entero positivo (de 1 a 1024)
* *delimitador*: una expresión de cadena opcional (predeterminado: cadena vacía)

**Devuelve**

Valor repetido para un número especificado de veces, concatenado con *delimitador*.

En caso de *que el multiplicador* sea superior al valor máximo permitido (1024), la cadena de entrada se repetirá 1024 veces.
 
**Ejemplo**

```kusto
print from_str = strrep('ABC', 2), from_int = strrep(123,3,'.'), from_time = strrep(3s,2,' ')
```

|from_str|from_int|from_time|
|---|---|---|
|ABCABC|123.123.123|00:00:03 00:00:03|