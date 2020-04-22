---
title: stdevp() (función de agregación) - Explorador de datos de Azure ( stdevp() (función de agregación) - Explorador de datos de Azure ( Stdevp() ( Microsoft Docs
description: En este artículo se describe stdevp() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0549c15ec9e2435d242f210e6dfc2163796e5f39
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663204"
---
# <a name="stdevp-aggregation-function"></a>stdevp() (función de agregación)

Calcula la desviación estándar de *Expr* en todo el grupo, considerando el grupo como una [población.](https://en.wikipedia.org/wiki/Statistical_population) 

* Fórmula usada:

:::image type="content" source="images/aggregations/stdev-population.png" alt-text="Población de Stdev":::

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

resumir `stdevp(` *Expr*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. 

**Devuelve**

El valor de desviación estándar de *Expr* en todo el grupo.
 
**Ejemplos**

```kusto
range x from 1 to 5 step 1
| summarize make_list(x), stdevp(x)

```

|list_x|stdevp_x|
|---|---|
|[ 1, 2, 3, 4, 5]|1.4142135623731|