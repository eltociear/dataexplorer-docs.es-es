---
title: 'hash_many (): Explorador de datos de Azure'
description: En este artículo se describe hash_many () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/06/2020
ms.openlocfilehash: 66ca1e5ff330a4b39ab769b0e3e8d6359eed9c00
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226676"
---
# <a name="hash_many"></a>hash_many()

Devuelve un valor hash combinado de varios valores.

**Sintaxis**

`hash_many(`*S1* `,` *S2* [ `,` *S3* ...]`)`

**Argumentos**

* *S1*, *S2*,..., *SN*: valores de entrada a los que se aplicará un algoritmo hash.

**Devuelve**

Valor hash combinado de los valores escalares especificados.

**Ejemplos**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print value1 = "Hello", value2 = "World"
| extend combined = hash_many(value1, value2)
```

|value1|value2|conjuntamente|
|---|---|---|
|Hola|World|-1440138333540407281|
