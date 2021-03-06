---
title: 'stdev () (función de agregación): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe stdev () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 3a29621a18a364145585022b1f0651100cadab1c
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82618551"
---
# <a name="stdev-aggregation-function"></a>stdev () (función de agregación)

Calcula la desviación estándar de *expr* en el grupo, teniendo en cuenta el grupo como [ejemplo](https://en.wikipedia.org/wiki/Sample_%28statistics%29). 

* Fórmula usada:

:::image type="content" source="images/stdev-aggfunction/stdev-sample.png" alt-text="Ejemplo de stdev":::

* Solo se puede usar en el contexto de la agregación dentro de [resumir](summarizeoperator.md)

**Sintaxis**

`stdev(`resumir *expr*`)`

**Argumentos**

* *Expr*: expresión que se utilizará para el cálculo de agregaciones. 

**Devuelve**

Valor de desviación estándar de *expr* en el grupo.
 
**Ejemplos**

```kusto
range x from 1 to 5 step 1
| summarize make_list(x), stdev(x)

```

|list_x|stdev_x|
|---|---|
|[1, 2, 3, 4, 5]|1.58113883008419|