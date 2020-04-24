---
title: 'Operador de serialización: Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador de serialización en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 5402d1e1fcceb42f02643bf24918ed07beddaed7
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509119"
---
# <a name="serialize-operator"></a>Operador serialize

Marca ese orden del conjunto de filas de entrada es seguro para el uso de funciones de ventana.

El operador tiene un significado declarativo y marca la fila de entrada establecida como serializada (ordenada) para que se le puedan aplicar funciones de [ventana.](./windowsfunctions.md)

```kusto
T | serialize rn=row_number()
```

**Sintaxis**

`serialize`[*Name1* `=` *Expr1* [`,` *Name2* `=` *Expr2*]...]

* Los pares *Name*/*Expr* son similares a los del [operador extend](./extendoperator.md).

**Ejemplo**

```kusto
Traces
| where ActivityId == "479671d99b7b"
| serialize

Traces
| where ActivityId == "479671d99b7b"
| serialize rn = row_number()
```

El conjunto de filas de salida de los siguientes operadores se marca como serializado:

[rango](./rangeoperator.md), [ordenar](./sortoperator.md), [orden](./orderoperator.md), [superior](./topoperator.md), [top-hitters](./tophittersoperator.md), [getschema](./getschemaoperator.md).

El conjunto de filas de salida de los siguientes operadores se marca como no serializado:

[sample](./sampleoperator.md), [sample-distinct](./sampledistinctoperator.md), [distinct](./distinctoperator.md), [join](./joinoperator.md), [top-nested](./topnestedoperator.md), [count](./countoperator.md), [summarize](./summarizeoperator.md), [facet](./facetoperator.md), [mv-expand](./mvexpandoperator.md), [evaluate](./evaluateoperator.md), [reduce by](./reduceoperator.md), [make-series](./make-seriesoperator.md)

Todos los demás operadores conservan la propiedad de serialización (si se serializa el conjunto de filas de entrada, al igual que el conjunto de filas de salida).