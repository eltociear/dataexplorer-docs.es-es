---
title: 'variancep () (función de agregación): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe variancep () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 80f3f900649d2c4c36c7a50831e011f0ee018860
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82618024"
---
# <a name="variancep-aggregation-function"></a>variancep () (función de agregación)

Calcula la varianza de *expr* en el grupo, teniendo en cuenta el grupo como una [población](https://en.wikipedia.org/wiki/Statistical_population). 

* Fórmula usada:

:::image type="content" source="images/variancep-aggfunction/variance-population.png" alt-text="Población de la varianza":::

* Solo se puede usar en el contexto de la agregación dentro de [resumir](summarizeoperator.md)

**Sintaxis**

`variancep(`resumir *expr*`)`

**Argumentos**

* *Expr*: expresión que se utilizará para el cálculo de agregaciones. 

**Devuelve**

Valor de varianza de *expr* en el grupo.
 
**Ejemplos**

```kusto
range x from 1 to 5 step 1
| summarize make_list(x), variancep(x) 
```

|list_x|variance_x|
|---|---|
|[1, 2, 3, 4, 5]|2|