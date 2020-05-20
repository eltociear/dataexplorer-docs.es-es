---
title: 'Funciones de ventana: Azure Explorador de datos'
description: En este artículo se describen las funciones de ventana de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/20/2019
ms.openlocfilehash: d876f26de796008e83b620e4511a31cdb4e23888
ms.sourcegitcommit: 974d5f2bccabe504583e387904851275567832e7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/18/2020
ms.locfileid: "83550697"
---
# <a name="window-functions"></a>Funciones de ventana

Las funciones de ventana operan en varias filas (registros) de un conjunto de filas a la vez. A diferencia de las funciones de agregación, las funciones de ventana requieren que las filas del conjunto de filas se serialicen (tienen un orden específico para ellas). Las funciones de ventana pueden depender del orden para determinar el resultado.

Las funciones de ventana solo se pueden usar en conjuntos serializados. La forma más fácil de serializar un conjunto de filas es usar el [operador Serialize](./serializeoperator.md). Este operador "inmoviliza" el orden de las filas de forma arbitraria. Si el orden de las filas serializadas es importante semánticamente, utilice el [operador Sort](./sortoperator.md) para forzar un orden determinado.

El proceso de serialización tiene asociado un costo no trivial. Por ejemplo, podría impedir el paralelismo de consultas en muchos escenarios. Por lo tanto, no aplique la serialización innecesariamente. Si es necesario, reorganice la consulta para realizar la serialización en el conjunto de filas más pequeño posible.

## <a name="serialized-row-set"></a>Conjunto de filas serializado

Un conjunto de filas arbitrario (como una tabla o el resultado de un operador tabular) se puede serializar de una de las siguientes maneras:

1. Ordenando el conjunto de filas. A continuación se muestra una lista de operadores que emiten conjuntos de filas ordenados.
2. Mediante el [operador Serialize](./serializeoperator.md).

Muchos operadores tabulares serializan la salida cada vez que la entrada ya está serializada, incluso si el operador no garantiza que el resultado se serialice. Por ejemplo, esta propiedad está garantizada para el [operador Extend](./extendoperator.md), el operador de [proyecto](./projectoperator.md)y el [operador Where](./whereoperator.md).

## <a name="operators-that-emit-serialized-row-sets-by-sorting"></a>Operadores que emiten conjuntos de filas serializados mediante la ordenación

* [Operador order](./orderoperator.md)
* [Operador sort](./sortoperator.md)
* [Operador top](./topoperator.md)
* [operador top-hitters](./tophittersoperator.md)
* [top-nested operator](./topnestedoperator.md)

## <a name="operators-that-preserve-the-serialized-row-set-property"></a>Operadores que conservan la propiedad del conjunto de filas serializado

* [Operador extend](./extendoperator.md)
* [Operador mv-expand](./mvexpandoperator.md)
* [Operador parse](./parseoperator.md)
* [Operador project](./projectoperator.md)
* [Operador project-away](./projectawayoperator.md)
* [Operador project-rename](./projectrenameoperator.md)
* [Operador take](./takeoperator.md)
* [Operador where](./whereoperator.md)
