---
title: bitset_count_ones() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe bitset_count_ones() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/22/2020
ms.openlocfilehash: c23e95ee9f00a0ca173d68d3591ad604b31dfac2
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517398"
---
# <a name="bitset_count_ones"></a>bitset_count_ones()

Devuelve el número de bits establecidos en la representación binaria de un número.

```kusto
bitset_count_ones(42)
```

**Sintaxis**

`bitset_count_ones(`*num1*'')'

**Argumentos**

* *num1*: número largo o entero.

**Devuelve**

Devuelve el número de bits establecidos en la representación binaria de un número.

**Ejemplo**

```kusto
// 42 = 32+8+2 : b'00101010' == 3 bits set
print ones = bitset_count_ones(42) 
```

|unos|
|---|
|3|