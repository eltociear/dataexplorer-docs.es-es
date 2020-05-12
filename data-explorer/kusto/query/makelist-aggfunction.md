---
title: 'make_list () (función de agregación): Explorador de datos de Azure | Microsoft Docs'
description: En este artículo se describe make_list () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/23/2020
ms.openlocfilehash: fed2616f5fbd32b1c80f936d5689261467a9dc5e
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83224841"
---
# <a name="make_list-aggregation-function"></a>make_list () (función de agregación)

Devuelve una matriz `dynamic` (JSON) de todos los valores de *Expr* del grupo.

* Solo se puede usar en el contexto de la agregación dentro de [resumir](summarizeoperator.md)

**Sintaxis**

`summarize``make_list(` *Expr* [ `,` *MaxSize*]`)`

**Argumentos**

* *Expr*: expresión que se utilizará para el cálculo de agregaciones.
* *MaxSize* es un límite entero opcional del número máximo de elementos devueltos (el valor predeterminado es *1048576*). El valor MaxSize no puede superar 1048576.

> [!NOTE]
> Una variante heredada y obsoleta de esta función: `makelist()` tiene un límite predeterminado de *MaxSize* = 128.

**Devuelve**

Devuelve una matriz `dynamic` (JSON) de todos los valores de *Expr* del grupo.
Si la entrada al `summarize` operador no está ordenada, el orden de los elementos de la matriz resultante es indefinido.
Si la entrada al `summarize` operador está ordenada, el orden de los elementos de la matriz resultante realiza un seguimiento de la entrada.

> [!TIP]
> Use el [`mv-apply`](./mv-applyoperator.md) operador para crear una lista ordenada por alguna clave. Puede consultar ejemplos [aquí](./mv-applyoperator.md#using-the-mv-apply-operator-to-sort-the-output-of-makelist-aggregate-by-some-key).

**Vea también**

[`make_list_if`](./makelistif-aggfunction.md)el operador es similar a `make_list` , salvo que también acepta un predicado.