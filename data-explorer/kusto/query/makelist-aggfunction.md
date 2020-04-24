---
title: make_list() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe make_list() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/23/2020
ms.openlocfilehash: e46dbfac7b8c819f66d469c160452ec4dddfb769
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512757"
---
# <a name="make_list-aggregation-function"></a>make_list() (función de agregación)

Devuelve una matriz `dynamic` (JSON) de todos los valores de *Expr* del grupo.

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

`summarize``make_list(` *Expr* `,` [ *MaxSize*]`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación.
* *MaxSize* es un límite entero opcional en el número máximo de elementos devueltos (el valor predeterminado es *1048576*). El valor MaxSize no puede superar 1048576.

> [!NOTE]
> Una variante heredada y obsoleta `makelist()` de esta función: tiene un límite predeterminado de *MaxSize* 128.

**Devuelve**

Devuelve una matriz `dynamic` (JSON) de todos los valores de *Expr* del grupo.
Si la entrada `summarize` del operador no está ordenada, el orden de los elementos de la matriz resultante es indefinido.
Si se ordena `summarize` la entrada al operador, el orden de los elementos de la matriz resultante realiza un seguimiento del de la entrada.

> [!TIP]
> Utilice [`mv-apply`](./mv-applyoperator.md) el operador para crear una lista ordenada con alguna clave. Puede consultar ejemplos [aquí](./mv-applyoperator.md#using-mv-apply-operator-to-sort-the-output-of-makelist-aggregate-by-some-key).

**Vea también**

[`make_list_if`](./makelistif-aggfunction.md)operador es `make_list`similar a , excepto que también acepta un predicado.