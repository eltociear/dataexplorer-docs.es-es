---
title: Funciones de ventana- Explorador de Azure Data Explorer Microsoft Docs
description: En este artículo se describen las funciones de Windows en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/20/2019
ms.openlocfilehash: ab4f6da2478ba4de81b2034c1cb07458daa80bd0
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81504257"
---
# <a name="window-functions"></a>Funciones de ventana

Las funciones de ventana funcionan en varias filas (registros) de un conjunto de filas a la vez.
A diferencia de las funciones de agregación, requieren que las filas del conjunto de filas se **serializan** (tienen un orden específico para ellas), ya que las funciones de ventana pueden depender del orden para determinar el resultado.

Las funciones de ventana no se pueden usar en conjuntos de filas que no se serializan y darán un error cuando se usan en un contexto de este tipo por una consulta. La forma más fácil de serializar un conjunto de filas es utilizar el [operador serialize](./serializeoperator.md), que simplemente "congela" el orden de las filas (de alguna manera arbitraria no especificada).
Si el orden de las filas serializadas es semánticamente importante, se puede utilizar el operador de [ordenación](./sortoperator.md) para forzar un orden determinado.

El proceso de serialización tiene un costo no trivial asociado. Por ejemplo, podría evitar el paralelismo de consultas en muchos escenarios. Por lo tanto, se recomienda encarecidamente que la serialización no se aplique innecesariamente y que, si es necesario, la consulta se reorganice para que la serialización se realice en el conjunto de filas más pequeño posible.

## <a name="serialized-row-set"></a>Conjunto de filas serializado

Un conjunto de filas arbitrario (como una tabla o la salida de un operador tabular) se puede serializar de una de las siguientes maneras:

1. Ordenando el conjunto de filas. Consulte a continuación una lista de operadores que emiten conjuntos de filas ordenados.
2. Mediante el [operador serialize](./serializeoperator.md).

Tenga en cuenta que muchos operadores tabulares, aunque en sí mismos no garantizan que su resultado se serializa, tienen la propiedad de que si se serializa la entrada, también lo es la salida. Por ejemplo, esta propiedad está garantizada para el [operador extend](./extendoperator.md), el operador de [proyecto](./projectoperator.md)y el [operador where](./whereoperator.md).

## <a name="operators-that-emit-serialized-row-sets-by-sorting"></a>Operadores que emiten conjuntos de filas serializados por ordenación

* [Operador order](./orderoperator.md)
* [Operador sort](./sortoperator.md)
* [operador superior](./topoperator.md)
* [operador top-hitters](./tophittersoperator.md)
* [top-nested operator](./topnestedoperator.md)

## <a name="operators-that-preserve-the-serialized-row-set-property"></a>Operadores que conservan la propiedad de conjunto de filas serializada

* [Operador extend](./extendoperator.md)
* [operador mv-expand](./mvexpandoperator.md)
* [Operador parse](./parseoperator.md)
* [Operador project](./projectoperator.md)
* [Operador project-away](./projectawayoperator.md)
* [Operador project-rename](./projectrenameoperator.md)
* [Operador take](./takeoperator.md)
* [donde el operador](./whereoperator.md)