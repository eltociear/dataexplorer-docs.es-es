---
title: parse_ipv4_mask ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe la función parse_ipv6_mask () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 99618c4d171e8ccade39ce50b9a0d46f9769f5f8
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84301238"
---
# <a name="parse_ipv6_mask"></a>parse_ipv6_mask ()
 
Convierte la cadena IPv6/IPv4 y la máscara de la máscara en una representación de cadena IPv6 canónica.

```kusto
parse_ipv6_mask("127.0.0.1", 24) == '0000:0000:0000:0000:0000:ffff:7f00:0000'
parse_ipv6_mask(":fe80::85d:e82c:9446:7994", 120) == 'fe80:0000:0000:0000:085d:e82c:9446:7900'
```

**Sintaxis**

`parse_ipv6_mask(`*`Expr`*`, `*`PrefixMask`*`)`

**Argumentos**

* *`Expr`*: Expresión de cadena que representa la dirección de red IPv6/IPv4 que se convertirá en representación IPv6 canónica. La cadena puede incluir la máscara de red mediante [la notación de prefijo IP](#ip-prefix-notation).
* *`PrefixMask`*: Entero de 0 a 128 que representa el número de bits más significativos que se tienen en cuenta.

## <a name="ip-prefix-notation"></a>Notación de prefijo IP

Las direcciones IP se pueden definir `IP-prefix notation` mediante el uso de un carácter de barra diagonal ( `/` ).
La dirección IP a la izquierda de la barra diagonal ( `/` ) es la dirección IP base. El número (1 a 127) a la derecha de la barra diagonal ( `/` ) es el número de 1 bit contiguo de la máscara de bits.

**Devuelve**

Si la conversión se realiza correctamente, el resultado será una cadena que representa una dirección de red IPv6 canónica.
Si la conversión no se realiza correctamente, el resultado será `null` .

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(ip_string:string, netmask:long)
[
 // IPv4 addresses
 '192.168.255.255',     120,  // 120-bit netmask is used
 '192.168.255.255/24',  124,  // 120-bit netmask is used, as IPv4 address doesn't use upper 8 bits
 '255.255.255.255', 128,  // 128-bit netmask is used
 // IPv6 addresses
 'fe80::85d:e82c:9446:7994', 128,     // 128-bit netmask is used
 'fe80::85d:e82c:9446:7994/120', 124, // 120-bit netmask is used
 // IPv6 with IPv4 notation
 '::192.168.255.255',    128,  // 128-bit netmask is used
 '::192.168.255.255/24', 128,  // 120-bit netmask is used, as IPv4 address doesn't use upper 8 bits
]
| extend ip6_canonical = parse_ipv6_mask(ip_string, netmask)
```

|ip_string|m|ip6_canonical|
|---|---|---|
|192.168.255.255|120|0000:0000:0000:0000:0000: ffff: c0a8: ff00|
|192.168.255.255/24|124|0000:0000:0000:0000:0000: ffff: c0a8: ff00|
|255.255.255.255|128|0000:0000:0000:0000:0000: ffff: ffff: ffff|
|fe80:: 85d: e82c: 9446:7994|128|fe80:0000:0000:0000:085d: e82c: 9446:7994|
|fe80:: 85d: e82c: 9446:7994/120|124|fe80:0000:0000:0000:085d: e82c: 9446:7900|
|:: 192.168.255.255|128|0000:0000:0000:0000:0000: ffff: c0a8: ffff|
|:: 192.168.255.255/24|128|0000:0000:0000:0000:0000: ffff: c0a8: ff00|

## <a name="next-steps"></a>Pasos siguientes

Para otras funciones similares, vea:

* [parse_ipv4_mask()](parse-ipv4-maskfunction.md)
* [parse_ipv6 ()](parse-ipv6function.md)
