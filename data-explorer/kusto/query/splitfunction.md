---
title: split() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe split() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 6e3935c524056afd27eb0d5e2d80925e072c8fa7
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81507470"
---
# <a name="split"></a>split()

Divide una cadena determinada según un delimitador determinado y devuelve una matriz de cadenas con las subcadenas contenidas.

Opcionalmente, se puede devolver una subcadena específica si existe.

```kusto
split("aaa_bbb_ccc", "_") == ["aaa","bbb","ccc"]
```

**Sintaxis**

`split(`*source*`,` *delimitador de* fuente`,` [ *requestedIndex*]`)`

**Argumentos**

* *source*: La cadena de origen que se dividirá según el delimitador dado.
* *delimiter*: El delimitador que se usará para dividir la cadena de origen.
* *requestedIndex*: un índice opcional de base cero `int`. Si se proporciona, la matriz de cadenas devuelta contendrá la subcadena solicitada, si existe. 

**Devuelve**

Una matriz de cadenas que contiene las subcadenas de la cadena de origen determinada delimitadas por el delimitador especificado.

**Ejemplos**

```kusto
print
    split("aa_bb", "_"),           // ["aa","bb"]
    split("aaa_bbb_ccc", "_", 1),  // ["bbb"]
    split("", "_"),                // [""]
    split("a__b", "_"),            // ["a","","b"]
    split("aabbcc", "bb")          // ["aa","cc"]
```