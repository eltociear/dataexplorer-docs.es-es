---
title: 'operador Serialize: Azure Explorador de datos'
description: En este artículo se describe el operador Serialize en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 708a5ccd5f8402dedb074a6ab8c17b1d7762839c
ms.sourcegitcommit: ae72164adc1dc8d91ef326e757376a96ee1b588d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2020
ms.locfileid: "84717128"
---
# <a name="serialize-operator"></a>Operador serialize

Marca que el orden del conjunto de filas de entrada es seguro para las funciones de ventana.

El operador tiene un significado declarativo. Marca el conjunto de filas de entrada como serializado (ordenado), de modo que se le pueden aplicar [funciones de ventana](./windowsfunctions.md) .

```kusto
T | serialize rn=row_number()
```

**Sintaxis**

`serialize`[*Nombre1* `=` *Expr1* [ `,` *nombre2* `=` *expr2*]...]

* Los *Name* / pares de nombre*expr* son similares a los pares del [operador Extend](./extendoperator.md).

**Ejemplo**

```kusto
Traces
| where ActivityId == "479671d99b7b"
| serialize

Traces
| where ActivityId == "479671d99b7b"
| serialize rn = row_number()
```

El conjunto de filas de salida de los siguientes operadores está marcado como serializado.

[Range](./rangeoperator.md), [Sort](./sortoperator.md), [Order](./orderoperator.md), [Top](./topoperator.md), [Top-Hitters](./tophittersoperator.md), [GetSchema](./getschemaoperator.md).

El conjunto de filas de salida de los siguientes operadores está marcado como no serializado.

[Sample](./sampleoperator.md), [Sample-DISTINCT](./sampledistinctoperator.md), [DISTINCT](./distinctoperator.md), [join](./joinoperator.md), [Top-Nested](./topnestedoperator.md), [Count](./countoperator.md), [resume](./summarizeoperator.md), [Facet](./facetoperator.md), [MV-Expand](./mvexpandoperator.md), [Evaluate](./evaluateoperator.md), [reduce by](./reduceoperator.md), [Make-series](./make-seriesoperator.md)

Todos los demás operadores conservan la propiedad de serialización. Si se serializa el conjunto de filas de entrada, se serializa también el conjunto de filas de salida.
