---
title: 'parse_ipv6 (): Explorador de datos de Azure'
description: En este artículo se describe la función parse_ipv6 () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 05/27/2020
ms.openlocfilehash: d7858a67719bbde9a1ecedee5888abc7d0662e1e
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84301221"
---
# <a name="parse_ipv6"></a>parse_ipv6 ()

Convierte la cadena IPv6 o IPv4 en una representación de cadena IPv6 canónica.

```kusto
parse_ipv6("127.0.0.1") == '0000:0000:0000:0000:0000:ffff:7f00:0001'
parse_ipv6(":fe80::85d:e82c:9446:7994") == 'fe80:0000:0000:0000:085d:e82c:9446:7994'
```

**Sintaxis**

`parse_ipv6(`*`Expr`*`)`

**Argumentos**

* *`Expr`*: Expresión de cadena que representa la dirección de red IPv6/IPv4 que se convertirá en representación IPv6 canónica. La cadena puede incluir la máscara de red mediante [la notación de prefijo IP](#ip-prefix-notation).

## <a name="ip-prefix-notation"></a>Notación de prefijo IP

Las direcciones IP se pueden definir `IP-prefix notation` mediante el uso de un carácter de barra diagonal ( `/` ).
La dirección IP a la izquierda de la barra diagonal ( `/` ) es la dirección IP base. El número (de 1 a 127) a la derecha de la barra diagonal ( `/` ) es el número de 1 bits contiguos de la máscara de bits.

**Devuelve**

Si la conversión se realiza correctamente, el resultado será una cadena que representa una dirección de red IPv6 canónica.
Si la conversión no se realiza correctamente, el resultado será `null` .

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(ip_string:string, netmask:long)
[
 '192.168.255.255',     32,  // 32-bit netmask is used
 '192.168.255.255/24',  30,  // 24-bit netmask is used, as IPv4 address doesn't use upper 8 bits
 '255.255.255.255',     24,  // 24-bit netmask is used
]
| extend ip_long = parse_ipv4_mask(ip_string, netmask)
```

|ip_string|m|ip_long|
|---|---|---|
|192.168.255.255|32|3232301055|
|192.168.255.255/24|30|3232300800|
|255.255.255.255|24|4294967040|

## <a name="next-steps"></a>Pasos siguientes

Para otras funciones similares, vea:

* [parse_ipv4()](parse-ipv4function.md)
* [ipv6_compare ()](ipv6-comparefunction.md)
* [ipv6_is_match ()](ipv6-is-matchfunction.md)
