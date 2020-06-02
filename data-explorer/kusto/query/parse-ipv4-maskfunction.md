---
title: parse_ipv4_mask ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe la función parse_ipv4_mask () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 05/27/2020
ms.openlocfilehash: fd6bd97befc376d13573a0a85169524cf01b9d2d
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84294617"
---
# <a name="parse_ipv4_mask"></a>parse_ipv4_mask()

Convierte la cadena de entrada de IPv4 y la máscara de la máscara en una representación de número largo (con signo 64 bits).

```kusto
parse_ipv4_mask("127.0.0.1", 24) == 2130706432
parse_ipv4_mask('192.1.168.2', 31) == parse_ipv4_mask('192.1.168.3', 31)
```

**Sintaxis**

`parse_ipv4_mask(`*`Expr`*`, `*`PrefixMask`*`)`

**Argumentos**

* *`Expr`*: Una representación de cadena de la dirección IPv4 que se convertirá en Long. 
* *`PrefixMask`*: Entero de 0 a 32 que representa el número de bits más significativos que se tienen en cuenta.

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un número largo.
Si la conversión no se realiza correctamente, el resultado será `null` .
