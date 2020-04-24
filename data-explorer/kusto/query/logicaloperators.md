---
title: 'Operadores lógicos (binarios): Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen los operadores lógicos (binarios) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/14/2018
ms.openlocfilehash: 53505067b93234aa89849e1c66fe2f77a58e0f26
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513097"
---
# <a name="logical-binary-operators"></a>Operadores lógicos (binarios)

Los siguientes operadores lógicos se `bool` admiten entre dos valores del tipo:

> [!NOTE]
> Estos operadores lógicos a veces se conocen como operadores booleanos y, a veces, como operadores binarios. Los nombres son todos sinónimos.

|Nombre del operador|Sintaxis|Significado|
|-------------|------|-------|
|Igualdad     |`==`  |Rendimientos `true` si ambos operandos no son nulos e iguales entre sí. En caso contrario, es `false`.|
|Desigualdad   |`!=`  |Rendimientos `true` si uno (o ambos) de los operandos son nulos o no son iguales entre sí. En caso contrario, es `true`.|
|Lógico y  |`and` |Rendimientos `true` si ambos `true`operandos son .|
|Lógico o   |`or`  |Rendimientos `true` si uno de `true`los operandos es , independientemente del otro operando.|

> [!NOTE]
> Debido al comportamiento del valor `bool(null)`nulo booleano , dos valores booleanos nulos `bool(null) != bool(null)` no son `false`iguales ni iguales (en otras palabras, `bool(null) == bool(null)` y ambos producen el valor).
>
> Por `and` / `or` otro lado, trate el valor `false`nulo `bool(null) or true` como equivalente a , so is `true`, y `bool(null) and true` es `false`.