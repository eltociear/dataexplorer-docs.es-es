---
title: hll_merge() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe hll_merge() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/15/2019
ms.openlocfilehash: 4700d5c87bf0f29f7bba86d56114a6a61092da94
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514117"
---
# <a name="hll_merge-aggregation-function"></a>hll_merge() (función de agregación)

Combina los resultados de HLL en todo el grupo en un solo valor HLL.

* Solo se puede utilizar en el contexto de la agregación dentro de [resumir](summarizeoperator.md).

Lea sobre el [algoritmo subyacente (*H*yper*L*og*L*og) y](dcount-aggfunction.md#estimation-accuracy)la precisión de estimación .

**Sintaxis**

`summarize``hll_merge(` *Expr*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. 

**Devuelve**

Los valores hll combinados de *Expr* en todo el grupo.
 
**Sugerencias**

1) Puede utilizar la función [dcount_hll] (dcount-hllfunction.md) que calculará el dcount a partir de funciones de agregación hll / hll-merge.