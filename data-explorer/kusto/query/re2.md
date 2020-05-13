---
title: 'Expresiones regulares: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las expresiones regulares de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/09/2019
ms.openlocfilehash: 3bbd14031adbfee3b5fac07194f5ff879ff33693
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373076"
---
# <a name="regular-expressions"></a>Expresiones regulares

RE2 sintaxis de expresiones regulares describe la sintaxis de la biblioteca de expresiones regulares usada por Kusto (RE2).
Hay algunas funciones en Kusto que realizan la coincidencia de cadenas, la selección y la extracción mediante una expresión regular.

- [countof()](countoffunction.md)
- [extract()](extractfunction.md)
- [extract_all()](extractallfunction.md)
- [matches regex](datatypes-string-operators.md)
- [Operador parse](parseoperator.md)
- [reemplazar ()](replacefunction.md)
- [Trim ()](trimfunction.md)
- [TrimEnd ()](trimendfunction.md)
- [TrimStart ()](trimstartfunction.md)

La sintaxis de expresiones regulares admitida por Kusto es la de la [biblioteca RE2](https://github.com/google/re2/wiki/Syntax). Estas expresiones se deben codificar en Kusto como `string` literales y se aplican todas las reglas de Comillas de cadenas de Kusto. Por ejemplo, la expresión regular `\A` coincide con el principio de una línea y se especifica en Kusto como literal de cadena `"\\A"` (observe el carácter de barra diagonal inversa "extra" `\` ).
