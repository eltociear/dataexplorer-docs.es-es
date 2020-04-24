---
title: parse_ipv4_mask() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe parse_ipv4_mask() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 994c1e551d7c38bb1493579bcf10438acd2923f2
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511788"
---
# <a name="parse_ipv4_mask"></a>parse_ipv4_mask()

Convierte la cadena de entrada de IPv4 y netmask en representación de número largo (firmado de 64 bits).

```kusto
parse_ipv4_mask("127.0.0.1", 24) == 2130706432

parse_ipv4_mask('192.1.168.2', 31) == parse_ipv4_mask('192.1.168.3', 31) 
```

**Sintaxis**

`parse_ipv4_mask(`*Expr*`, `*PrefixMask*`)`

**Argumentos**

* *Expr*: Una representación de cadena de la dirección IPv4 que se convertirá a long. 
* *PrefixMask*: Un entero de 0 a 32 que representa el número de bits más significativos que se tienen en cuenta.

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un número largo.
Si la conversión no se `null`realiza correctamente, el resultado será .