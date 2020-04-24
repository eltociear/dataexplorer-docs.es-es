---
title: substring() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe substring() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: b0273b3e93c8778af9c380f164faec74349aa8cd
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506705"
---
# <a name="substring"></a>substring()

Extrae una subcadena de una cadena de origen a partir de algún índice hasta el final de la cadena.

Opcionalmente, se puede especificar la longitud de la subcadena solicitada.

```kusto
substring("abcdefg", 1, 2) == "bc"
```

**Sintaxis**

`substring(`*fuente* `,` *startingIndex* [`,` *length*]`)`

**Argumentos**

* *source*: La cadena de origen de la que se tomará la subcadena.
* *startingIndex*: La posición de carácter inicial de base cero de la subcadena solicitada.
* *longitud*: un parámetro opcional que se puede utilizar para especificar el número solicitado de caracteres en la subcadena. 

**Notas**

*startingIndex* puede ser un número negativo, en cuyo caso la subcadena se recuperará del final de la cadena de origen.

**Devuelve**

Una subcadena de la cadena especificada. La subcadena comienza en la posición del carácter startingIndex (basado en cero) y continúa hasta el final de los caracteres de cadena o longitud, si se especifican.

**Ejemplos**

```kusto
substring("123456", 1)        // 23456
substring("123456", 2, 2)     // 34
substring("ABCD", 0, 2)       // AB
substring("123456", -2, 2)    // 56
```