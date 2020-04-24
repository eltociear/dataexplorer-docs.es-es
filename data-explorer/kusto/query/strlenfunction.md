---
title: strlen() - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe strlen() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 33bb1ea56ebc03dc7357264bcdf987c191478cbb
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506875"
---
# <a name="strlen"></a>strlen()

Devuelve la longitud, en caracteres, de la cadena de entrada.

**Sintaxis**

`strlen(`*Fuente*`)`

**Argumentos**

* *source*: La cadena de origen que se medirá para la longitud de la cadena.

**Devuelve**

Devuelve la longitud, en caracteres, de la cadena de entrada.

**Notas**

Cada carácter Unicode de `1`la cadena es igual a , incluidos los sustitutos.
(por ejemplo: los caracteres chinos se contarán una vez a pesar de que requiere más de un valor en la codificación UTF-8).


**Ejemplos**

```kusto
print length = strlen("hello")
```

|length|
|---|
|5|

```kusto
print length = strlen("⒦⒰⒮⒯⒪")
```

|length|
|---|
|5|