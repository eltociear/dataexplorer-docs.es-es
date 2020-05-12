---
title: 'make_list_with_nulls () (función de agregación): Explorador de datos de Azure | Microsoft Docs'
description: En este artículo se describe make_list_with_nulls () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/09/2020
ms.openlocfilehash: 41f07f16641fd303c9b8e76b4924238378b6ccc9
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83224823"
---
# <a name="make_list_with_nulls-aggregation-function"></a>make_list_with_nulls () (función de agregación)

Devuelve una `dynamic` matriz (JSON) de todos los valores de *expr* en el grupo, incluidos los valores NULL.

* Solo se puede usar en el contexto de la agregación dentro de [resumir](summarizeoperator.md)

**Sintaxis**

`summarize``make_list_with_nulls(` *Expr*`)`

**Argumentos**

* *Expr*: expresión que se utilizará para el cálculo de agregaciones.

**Devuelve**

Devuelve una `dynamic` matriz (JSON) de todos los valores de *expr* en el grupo, incluidos los valores NULL.
Si la entrada al `summarize` operador no está ordenada, el orden de los elementos de la matriz resultante es indefinido.
Si la entrada al `summarize` operador está ordenada, el orden de los elementos de la matriz resultante realiza un seguimiento de la entrada.

> [!TIP]
> Use el [`mv-apply`](./mv-applyoperator.md) operador para crear una lista ordenada por alguna clave. Puede consultar ejemplos [aquí](./mv-applyoperator.md#using-the-mv-apply-operator-to-sort-the-output-of-makelist-aggregate-by-some-key).
