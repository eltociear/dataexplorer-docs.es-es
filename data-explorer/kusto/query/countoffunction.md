---
title: countof() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe countof() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1d932fbcea9b38849e7d7de09230c9a5aa9fa8e4
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516905"
---
# <a name="countof"></a>countof()

Cuenta las apariciones de una subcadena en una cadena. Las coincidencias de cadena sin formato se pueden superponer; las coincidencias de expresión regular no lo hacen.

```kusto
countof("The cat sat on the mat", "at") == 3
countof("The cat sat on the mat", @"\b.at\b", "regex") == 3
```

**Sintaxis**

`countof(`*text*`,` *búsqueda* de`,` texto [ *tipo*]`)`

**Argumentos**

* *texto*: Una cadena.
* *buscar*: La cadena sin formato o [expresión regular](./re2.md) para que coincida con el *texto*interior .
* *tipo* `"normal"|"regex"` : `normal`Predeterminado . 

**Devuelve**

El número de veces que la cadena de búsqueda puede coincidir en el contenedor. Las coincidencias de cadena sin formato se pueden superponer; las coincidencias de expresión regular no lo hacen.

**Ejemplos**

|||
|---|---
|`countof("aaa", "a")`| 3 
|`countof("aaaa", "aa")`| 3 (¡no 2!)
|`countof("ababa", "ab", "normal")`| 2
|`countof("ababa", "aba")`| 2
|`countof("ababa", "aba", "regex")`| 1
|`countof("abcabc", "a.c", "regex")`| 2
    