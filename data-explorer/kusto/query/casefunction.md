---
title: case() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe case() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 479c4a99d410a2df7a608531914d7dccfc555e7e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517279"
---
# <a name="case"></a>case()

Evalúa una lista de predicados y devuelve la primera expresión de resultado cuyo predicado se cumple.

Si ninguno de los `true`predicados devuelve , `else`se devuelve el resultado de la última expresión (la ).
Todos los argumentos impares (el recuento comienza `boolean` en 1) deben ser expresiones que se evalúen como un valor.
Todos los argumentos `then`par (la s) `else`y el último argumento (el ) deben ser del mismo tipo.

**Sintaxis**

`case(`*then_1 predicate_1,* `,` *then_1* *then_2* *else* *predicate_2,* `,` *then_3* *predicate_3,* `,``)`

**Argumentos**

* *predicate_i:* una expresión que `boolean` se evalúa como un valor.
* *then_i*: una expresión que se evalúa y su *predicate_i* valor se devuelve desde la `true`función si predicate_i es el primer predicado que se evalúa como .
* *una*expresión que se evalúa y su valor se devuelve desde la función si ninguno de los *predicate_i* evaluar como `true`.

**Devuelve**

El valor de *then_i* la primera then_i `true`cuyo *predicate_i* se evalúa como , o el valor de *else* si no se cumple ninguno de los predicados.

**Ejemplo**

```kusto
range Size from 1 to 15 step 2
| extend bucket = case(Size <= 3, "Small", 
                       Size <= 10, "Medium", 
                       "Large")
```

|Tamaño|bucket|
|---|---|
|1|Pequeña|
|3|Pequeña|
|5|Media|
|7|Media|
|9|Media|
|11|grande|
|13|grande|
|15|grande|
