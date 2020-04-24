---
title: stdevif() (función de agregación) - Explorador de datos de Azure ( Stdevif() (función de agregación) - Explorador de datos de Azure ( Stdevif() Microsoft Docs
description: En este artículo se describe stdevif() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4a64cf1bb69860a2a8bd64de91cb00c2f0ec296f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506977"
---
# <a name="stdevif-aggregation-function"></a>stdevif() (función de agregación)

Calcula el [stdev](stdev-aggfunction.md) de *Expr* en el grupo `true`para el que *Predicate* se evalúa como .

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

resumir `stdevif(` *Expr*`, `*Predicate*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. 
* *Predicado*: predicado que si es true, el *Expr* valor calculado se agregará a la desviación estándar.

**Devuelve**

El valor de desviación estándar de *Expr* en todo el grupo donde *Predicate* se evalúa como `true`.
 
**Ejemplos**

```kusto
range x from 1 to 100 step 1
| summarize stdevif(x, x%2 == 0)

```

|stdevif_x|
|---|
|29.1547594742265|