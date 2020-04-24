---
title: replace() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe replace() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 84a741f10172ef418da7d92b8c1ad6ba26593d72
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510343"
---
# <a name="replace"></a>replace()

Reemplace todas las coincidencias de expresiones regulares por otra cadena.

**Sintaxis**

`replace(`*regex* `,` *reescribir* `,` *texto*`)`

**Argumentos**

* *regex*: La [expresión regular](https://github.com/google/re2/wiki/Syntax) para buscar *texto*. Puede contener grupos de captura entre "("paréntesis")". 
* *reescritura*: El regex de reemplazo para cualquier coincidencia realizada por *matchingRegex*. Use `\0` para hacer referencia a toda la coincidencia, `\1` para el primer grupo de capturas, `\2` y así sucesivamente para los grupos de capturas posteriores.
* *texto*: Una cadena.

**Devuelve**

*text* después de reemplazar todas las coincidencias de la *expresión regular* por evaluaciones de *rewrite*. Las coincidencias no se superponen.

**Ejemplo**

Esta instrucción:

```kusto
range x from 1 to 5 step 1
| extend str=strcat('Number is ', tostring(x))
| extend replaced=replace(@'is (\d+)', @'was: \1', str)
```

Tiene los resultados siguientes:

| x    | str | reemplazado|
|---|---|---|
| 1    | El número es 1.000000  | El número era 1.000000|
| 2    | El número es 2.000000  | El número era 2.000000|
| 3    | El número es 3.000000  | El número era 3.000000|
| 4    | El número es 4.000000  | El número era 4.000000|
| 5    | El número es 5.000000  | El número era 5.000000|
 