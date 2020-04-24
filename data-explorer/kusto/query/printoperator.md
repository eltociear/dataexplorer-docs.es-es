---
title: 'operador de impresión: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador de impresión en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/16/2019
ms.openlocfilehash: b751b07446a74ef17002abac26e4c881a2da757b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510972"
---
# <a name="print-operator"></a>Operador print

Produce una sola fila con una o más expresiones escalares.

```kusto
print x=1, s=strcat("Hello", ", ", "World!")
```

**Sintaxis**

`print`[*ColumnName* `=`] *ScalarExpression* [',' ...]

**Argumentos**

* *ColumnName*: un nombre de opción para asignar a la columna singular de la salida.
* *ScalarExpression*: Una expresión escalar para evaluar.

**Devuelve**

Una tabla de una sola columna, de una sola fila, cuya celda única tiene el valor de *ScalarExpression*evaluada .

**Ejemplos**

El `print` operador es útil como una forma rápida de evaluar una o más expresiones escalares y crear una tabla de una sola fila de los valores resultantes.
Por ejemplo:

```kusto
print 0 + 1 + 2 + 3 + 4 + 5, x = "Wow!"
```

```kusto
print banner=strcat("Hello", ", ", "World!")
```