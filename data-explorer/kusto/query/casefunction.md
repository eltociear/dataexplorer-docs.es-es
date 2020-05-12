---
title: 'caso (): Azure Explorador de datos'
description: En este artículo se describen las mayúsculas y minúsculas () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: b493f74472454649a557b7e3677b26af169413de
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227560"
---
# <a name="case"></a>case()

Evalúa una lista de predicados y devuelve la primera expresión de resultado cuyo predicado se cumple.

Si no se devuelve ninguno de los predicados `true` , se devuelve el resultado de la última expresión ( `else` ).
Todos los argumentos impares (Count empieza en 1) deben ser expresiones que se evalúen como un `boolean` valor.
Todos los argumentos pares ( `then` s) y el último argumento ( `else` ) deben ser del mismo tipo.

**Sintaxis**

`case(`*predicate_1* `,` *then_1*, *predicate_2* `,` *then_2*, *predicate_3* `,` *then_3*, *else*`)`

**Argumentos**

* *predicate_i*: expresión que se evalúa como un `boolean` valor.
* *then_i*: una expresión que se evalúa y su valor se devuelve desde la función si *predicate_i* es el primer predicado que se evalúa como `true` .
* *else*: una expresión que se evalúa y su valor se devuelve desde la función si ninguno de los *predicate_i* se evalúa como `true` .

**Devuelve**

Valor de la primera *then_i* cuyo *predicate_i* se evalúa como `true` , o el valor de *else* si no se cumple ninguno de los predicados.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
range Size from 1 to 15 step 2
| extend bucket = case(Size <= 3, "Small", 
                       Size <= 10, "Medium", 
                       "Large")
```

|Size|bucket|
|---|---|
|1|Pequeña|
|3|Pequeña|
|5|Media|
|7|Media|
|9|Media|
|11|Grande|
|13|Grande|
|15|Grande|
