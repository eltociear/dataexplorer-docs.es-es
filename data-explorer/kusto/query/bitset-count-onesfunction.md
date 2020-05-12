---
title: 'bitset_count_ones (): Explorador de datos de Azure'
description: En este artículo se describe bitset_count_ones () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/22/2020
ms.openlocfilehash: f8abb1683a2f15f012e9a9271681688c19901af0
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227609"
---
# <a name="bitset_count_ones"></a>bitset_count_ones()

Devuelve el número de bits establecidos en la representación binaria de un número.

```kusto
bitset_count_ones(42)
```

**Sintaxis**

`bitset_count_ones(`*NUM1*' ') '

**Argumentos**

* *NUM1*: número entero o largo.

**Devuelve**

Devuelve el número de bits establecidos en la representación binaria de un número.

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
// 42 = 32+8+2 : b'00101010' == 3 bits set
print ones = bitset_count_ones(42) 
```

|correctas|
|---|
|3|
