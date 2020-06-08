---
title: 'hll_merge () (función de agregación): Explorador de datos de Azure | Microsoft Docs'
description: En este artículo se describe hll_merge () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/15/2019
ms.openlocfilehash: 59c6f6a11b108cf6e74ceb59d3483ea1a95f7002
ms.sourcegitcommit: 188f89553b9d0230a8e7152fa1fce56c09ebb6d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2020
ms.locfileid: "84512391"
---
# <a name="hll_merge-aggregation-function"></a>hll_merge () (función de agregación)

Combina `HLL` los resultados del grupo en un valor único `HLL` .

* Solo se puede usar en el contexto de la agregación dentro de [resumir](summarizeoperator.md).

Para obtener más información, vea el [algoritmo subyacente (*H*Yper*l*og*l*OG) y la precisión de estimación](dcount-aggfunction.md#estimation-accuracy).

**Sintaxis**

`summarize``hll_merge(` *Expr*`)`

**Argumentos**

* `*Expr*`: Expresión que se utilizará para el cálculo de agregaciones.

**Devuelve**

La función devuelve los valores combinados `hll` de `*Expr*` en el grupo.
 
**Sugerencias**

1) Utilice la función [dcount_hll] (DCount-hllfunction.MD) para calcular el a `dcount` partir de `hll`  /  `hll-merge` las funciones de agregación.
