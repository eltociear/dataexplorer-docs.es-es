---
title: 'sumar.Si () (función de agregación): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe sumar.Si () (función de agregación) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 7d97d31b2fb97d5541400bc0605ee40e83807b62
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83371892"
---
# <a name="sumif-aggregation-function"></a>sumar.Si () (función de agregación)

Devuelve una suma de *expr* para la que el *predicado* se evalúa como `true` .

* Solo se puede usar en el contexto de la agregación dentro de [resumir](summarizeoperator.md)

También puede utilizar la función [SUM ()](sum-aggfunction.md) , que suma las filas sin expresión de predicado.

**Sintaxis**

resumir el `sumif(` *Expr* `,` *predicado* expr`)`

**Argumentos**

* *Expr*: expresión para el cálculo de agregaciones. 
* *Predicado*: Predicate que, si es true, el valor calculado de *expr*se agregará a la suma. 

**Devuelve**

Valor de suma de *expr* para el que el *predicado* se evalúa como `true` .

**Ejemplo**

```kusto
let T = datatable(name:string, day_of_birth:long)
[
   "John", 9,
   "Paul", 18,
   "George", 25,
   "Ringo", 7
];
T
| summarize sumif(day_of_birth, strlen(name) > 4)
```

|sumif_day_of_birth|
|----|
|32|