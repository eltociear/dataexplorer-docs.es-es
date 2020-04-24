---
title: arg_max() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe arg_max() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 73953a17b1819081c8458d5dbde3fa6d55c8866e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518962"
---
# <a name="arg_max-aggregation-function"></a>arg_max() (función de agregación)

Busca una fila en el grupo que maximiza *ExprToMaximize*y devuelve el `*` valor de *ExprToReturn* (o para devolver toda la fila).

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

`summarize`[`(`*NameExprToMaximize* `,` *NameExprToReturn* [`,` ...] `)=` `arg_max` ] `(` *ExprToMaximize*, `*`  |  *ExprToReturn* [`,` ...]`)`

**Argumentos**

* *ExprToMaximize*: Expresión que se utilizará para el cálculo de agregación. 
* *ExprToReturn*: Expresión que se usará para devolver el valor cuando *ExprToMaximize* sea máximo. La expresión que se va a devolver puede ser un carácter comodín (*) para devolver todas las columnas de la tabla de entrada.
* *NameExprToMaximize*: Un nombre opcional para la columna de resultados que representa *ExprToMaximize*.
* *NameExprToReturn*: Nombres opcionales adicionales para las columnas de resultados que representan *ExprToReturn*.

**Devuelve**

Busca una fila en el grupo que maximiza *ExprToMaximize*y devuelve el `*` valor de *ExprToReturn* (o para devolver toda la fila).

**Ejemplos**

Vea ejemplos de la función de agregación [arg_min()](arg-min-aggfunction.md)