---
title: hash_many() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe hash_many() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/06/2020
ms.openlocfilehash: e98f9d1d956d15cd7a61e7873f9b1dd34c6ae288
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514185"
---
# <a name="hash_many"></a>hash_many()

Devuelve un valor hash combinado de varios valores.

**Sintaxis**

`hash_many(`*s1* `,` *s2* [`,` *s3* ...]`)`

**Argumentos**

* *s1*, *s2*, ..., *sN*: valores de entrada que se aplicarán hash juntos.

**Devuelve**

El valor hash combinado de los escalares especificados.

**Ejemplos**

```kusto
print value1 = "Hello", value2 = "World"
| extend combined = hash_many(value1, value2)
```

|value1|value2|Combinado|
|---|---|---|
|Hola|World|-1440138333540407281|