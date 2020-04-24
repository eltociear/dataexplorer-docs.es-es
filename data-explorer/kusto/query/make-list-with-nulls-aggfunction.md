---
title: make_list_with_nulls() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe make_list_with_nulls() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/09/2020
ms.openlocfilehash: 4b039008c5969cf02187d69a3486b09e04ec41ae
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512876"
---
# <a name="make_list_with_nulls-aggregation-function"></a>make_list_with_nulls() (función de agregación)

Devuelve `dynamic` una matriz (JSON) de todos los valores de *Expr* en el grupo, incluidos los valores nulos.

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

`summarize``make_list_with_nulls(` *Expr*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación.

**Devuelve**

Devuelve `dynamic` una matriz (JSON) de todos los valores de *Expr* en el grupo, incluidos los valores nulos.
Si la entrada `summarize` del operador no está ordenada, el orden de los elementos de la matriz resultante es indefinido.
Si se ordena `summarize` la entrada al operador, el orden de los elementos de la matriz resultante realiza un seguimiento del de la entrada.

> [!TIP]
> Utilice [`mv-apply`](./mv-applyoperator.md) el operador para crear una lista ordenada con alguna clave. Puede consultar ejemplos [aquí](./mv-applyoperator.md#using-mv-apply-operator-to-sort-the-output-of-makelist-aggregate-by-some-key).
