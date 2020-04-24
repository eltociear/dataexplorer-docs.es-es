---
title: hash_combine() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe hash_combine() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/19/2019
ms.openlocfilehash: d0c6375436df298a97c03ec2f06f7cd492d59d7f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514270"
---
# <a name="hash_combine"></a>hash_combine()

Combina valores hash de dos o más hashes.

**Sintaxis**

`hash_combine(`*h1* `,` *h2* [`,` *h3* ...]`)`

**Argumentos**

* *h1*: Valor largo que representa el primer valor hash.
* *h2*: Valor largo que representa el segundo valor hash.
* *hN*: Valor largo que representa el valor hash Nth.

**Devuelve**

El valor hash combinado de los escalares especificados.

**Ejemplos**

```kusto
print value1 = "Hello", value2 = "World"
| extend h1 = hash(value1), h2=hash(value2)
| extend combined = hash_combine(h1, h2)
```

|value1|value2|h1|h2|Combinado|
|---|---|---|---|---|
|Hola|World|753694413698530628|1846988464401551951|-1440138333540407281|