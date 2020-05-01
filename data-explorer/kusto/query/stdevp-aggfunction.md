---
title: 'STDEVP () (función de agregación): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe STDEVP () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: baafca4d8d5711d55838bceae817c36ecb0edd6f
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82618585"
---
# <a name="stdevp-aggregation-function"></a>STDEVP () (función de agregación)

Calcula la desviación estándar de *expr* en el grupo, teniendo en cuenta el grupo como una [población](https://en.wikipedia.org/wiki/Statistical_population). 

* Fórmula usada:

:::image type="content" source="images/stdevp-aggfunction/stdev-population.png" alt-text="Rellenado stdev":::

* Solo se puede usar en el contexto de la agregación dentro de [resumir](summarizeoperator.md)

**Sintaxis**

`stdevp(`resumir *expr*`)`

**Argumentos**

* *Expr*: expresión que se utilizará para el cálculo de agregaciones. 

**Devuelve**

Valor de desviación estándar de *expr* en el grupo.
 
**Ejemplos**

```kusto
range x from 1 to 5 step 1
| summarize make_list(x), stdevp(x)

```

|list_x|stdevp_x|
|---|---|
|[1, 2, 3, 4, 5]|1.4142135623731|