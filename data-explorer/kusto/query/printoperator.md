---
title: 'operador de impresión: Azure Explorador de datos'
description: En este artículo se describe el operador de impresión en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/16/2019
ms.openlocfilehash: d5788ee937fe110b63a8f137fdab0790eb7cb37e
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373201"
---
# <a name="print-operator"></a>Operador print

Genera una fila única con una o más expresiones escalares.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print x=1, s=strcat("Hello", ", ", "World!")
```

**Sintaxis**

`print`[*ColumnName* `=` ] *ScalarExpression* [', '...]

**Argumentos**

* *ColumnName*: nombre de la opción que se va a asignar a la columna singular de la salida.
* *ScalarExpression*: expresión escalar que se va a evaluar.

**Devuelve**

Tabla de una sola fila y una sola columna cuya celda única tiene el valor de *ScalarExpression*evaluado.

**Ejemplos**

El `print` operador es útil como una forma rápida de evaluar una o varias expresiones escalares y hacer que una tabla de fila única fuera de los valores resultantes.
Por ejemplo:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print 0 + 1 + 2 + 3 + 4 + 5, x = "Wow!"
```
<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print banner=strcat("Hello", ", ", "World!")
```
