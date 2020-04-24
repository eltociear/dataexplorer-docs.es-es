---
title: tdigest_merge() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe tdigest_merge() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/09/2019
ms.openlocfilehash: 0b7de916dd53c19a49301c8048e2d8867d1b1249
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506399"
---
# <a name="tdigest_merge-aggregation-function"></a>tdigest_merge() (función de agregación)

Combina los resultados de tdigest en todo el grupo. 

* Solo se puede utilizar en el contexto de la agregación dentro de [resumir](summarizeoperator.md).

Lea más sobre el algoritmo subyacente (T-Digest) y el error estimado [aquí](percentiles-aggfunction.md#estimation-error-in-percentiles).

**Sintaxis**

resumir `tdigest_merge(` *Expr*`)`.

resumir `tdigest_merge(` *Expr* `)` - Un alias.

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. 

**Devuelve**

Los valores tdigest combinados de *Expr* en todo el grupo.
 

**Sugerencias**

1) Puede utilizar la`percentile_tdigest()`función [ ] (percentile-tdigestfunction.md).

2) Todos los tdigests que se incluyen en el mismo grupo deben ser del mismo tipo.