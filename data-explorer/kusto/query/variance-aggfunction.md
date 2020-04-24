---
title: variance() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe variance() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 5b1d2ea47060ecede855a3fb419bbbfe2bf0b538
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81504716"
---
# <a name="variance-aggregation-function"></a>variance() (función de agregación)

Calcula la varianza de *Expr* en todo el grupo, considerando el grupo como una [muestra.](https://en.wikipedia.org/wiki/Sample_%28statistics%29) 

* Fórmula usada: ![texto alternativo](./images/aggregations/variance-sample.png "varianza-muestra")

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

resumir `variance(` *Expr*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. 

**Devuelve**

El valor de varianza de *Expr* en todo el grupo.
 
**Ejemplos**

```kusto
range x from 1 to 5 step 1
| summarize make_list(x), variance(x) 
```

|list_x|variance_x|
|---|---|
|[ 1, 2, 3, 4, 5]|2.5|