---
title: 'varianza () (función de agregación): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la varianza () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 9f8afae2413d65618cf6b6e2f400df2500b06078
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82618059"
---
# <a name="variance-aggregation-function"></a>Variance () (función de agregación)

Calcula la varianza de *expr* en el grupo, teniendo en cuenta el grupo como [ejemplo](https://en.wikipedia.org/wiki/Sample_%28statistics%29). 

* Fórmula usada:

:::image type="content" source="images/variance-aggfunction/variance-sample.png" alt-text="Ejemplo de varianza":::

* Solo se puede usar en el contexto de la agregación dentro de [resumir](summarizeoperator.md)

**Sintaxis**

`variance(`resumir *expr*`)`

**Argumentos**

* *Expr*: expresión que se utilizará para el cálculo de agregaciones. 

**Devuelve**

Valor de varianza de *expr* en el grupo.
 
**Ejemplos**

```kusto
range x from 1 to 5 step 1
| summarize make_list(x), variance(x) 
```

|list_x|variance_x|
|---|---|
|[1, 2, 3, 4, 5]|2.5|