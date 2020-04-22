---
title: stdev() (función de agregación) - Explorador de datos de Azure ( Stdev() (función de agregación) - Explorador de datos de Azure ( Stdev() Microsoft Docs
description: En este artículo se describe stdev() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: d54af583db6f7aca0b436040c453249207a5ae59
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663089"
---
# <a name="stdev-aggregation-function"></a>stdev() (función de agregación)

Calcula la desviación estándar de *Expr* en todo el grupo, considerando el grupo como una [muestra.](https://en.wikipedia.org/wiki/Sample_%28statistics%29) 

* Fórmula usada:

:::image type="content" source="images/aggregations/stdev-sample.png" alt-text="Muestra Stdev":::

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

resumir `stdev(` *Expr*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. 

**Devuelve**

El valor de desviación estándar de *Expr* en todo el grupo.
 
**Ejemplos**

```kusto
range x from 1 to 5 step 1
| summarize make_list(x), stdev(x)

```

|list_x|stdev_x|
|---|---|
|[ 1, 2, 3, 4, 5]|1.58113883008419|