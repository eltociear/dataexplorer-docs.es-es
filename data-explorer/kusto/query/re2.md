---
title: Expresiones regulares- Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describen las expresiones regulares en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/09/2019
ms.openlocfilehash: 0bc5dc6705d6cada446716f2f9d84618322ecd76
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510530"
---
# <a name="regular-expressions"></a>Expresiones regulares

La sintaxis de expresión regular RE2 describe la sintaxis de la biblioteca de expresiones regulares utilizada por Kusto (re2).
Hay algunas funciones en Kusto realizar la coincidencia de cadenas, selección y extracción mediante una expresión regular

- [countof()](countoffunction.md)
- [extracto()](extractfunction.md)
- [extract_all()](extractallfunction.md)
- [matches regex](datatypes-string-operators.md)
- [Operador parse](parseoperator.md)
- [reemplazar()](replacefunction.md)
- [ajuste()](trimfunction.md)
- [trimend()](trimendfunction.md)
- [trimstart()](trimstartfunction.md)

La sintaxis de expresión regular admitida por Kusto es la de la [biblioteca re2](https://github.com/google/re2/wiki/Syntax)y se detalla a continuación. Tenga en cuenta que estas expresiones `string` deben codificarse en Kusto como literales y se aplican todas las reglas de cita de cadena de Kusto. Por ejemplo, la `\A` expresión regular coincide con el principio de una línea (consulte la `"\\A"` tabla siguiente) y se`\`especifica en Kusto como el literal de cadena (tenga en cuenta el carácter de barra diagonal inversa "extra"( ) .)

