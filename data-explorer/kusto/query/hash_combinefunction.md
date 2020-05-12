---
title: 'hash_combine (): Explorador de datos de Azure'
description: En este artículo se describe hash_combine () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/19/2019
ms.openlocfilehash: 1925d9b27382dd3a888e14243bfecad51d37db0d
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226710"
---
# <a name="hash_combine"></a>hash_combine()

Combina los valores hash de dos o más hashes.

**Sintaxis**

`hash_combine(`*H1* `,` *H2* [ `,` *H3* ...]`)`

**Argumentos**

* *H1*: valor largo que representa el primer valor hash.
* *H2*: valor largo que representa el segundo valor hash.
* *hN*: valor Long que representa el valor de hash nth.

**Devuelve**

Valor hash combinado de los valores escalares especificados.

**Ejemplos**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
print value1 = "Hello", value2 = "World"
| extend h1 = hash(value1), h2=hash(value2)
| extend combined = hash_combine(h1, h2)
```

|value1|value2|H1|m2|conjuntamente|
|---|---|---|---|---|
|Hola|World|753694413698530628|1846988464401551951|-1440138333540407281|
