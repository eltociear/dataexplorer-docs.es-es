---
title: varianceif() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe varianceif() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 9dfebb3796f07dec6c91d36d788a018f84f70961
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81504682"
---
# <a name="varianceif-aggregation-function"></a>varianceif() (función de agregación)

Calcula la [varianza](variance-aggfunction.md) de *Expr* en el grupo `true`para el que *Predicate* se evalúa como .

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

resumir `varianceif(` *Expr*`, `*Predicate*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. 
* *Predicado*: predicado que si es true, el *Expr* valor calculado se agregará a la varianza.

**Devuelve**

El valor de varianza de *Expr* en `true`todo el grupo donde *Predicate* se evalúa como .
 
**Ejemplos**

```kusto
range x from 1 to 100 step 1
| summarize varianceif(x, x%2 == 0)

```

|varianceif_x|
|---|
|850|