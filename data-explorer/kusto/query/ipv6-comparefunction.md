---
title: 'ipv6_compare (): Explorador de datos de Azure'
description: En este artículo se describe la función ipv6_compare () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 05/27/2020
ms.openlocfilehash: 7d63ce48ba54377fa79ccd13484b2b9b08794bc6
ms.sourcegitcommit: 188f89553b9d0230a8e7152fa1fce56c09ebb6d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2020
ms.locfileid: "84512374"
---
# <a name="ipv6_compare"></a>ipv6_compare()

Compara dos cadenas de dirección de red IPv6 o IPv4. Las dos cadenas IPv6 se analizan y se comparan mientras se cuenta la máscara de prefijo de IP combinada calculada a partir de los prefijos de argumento y el `PrefixMask` argumento opcional.

```kusto
ipv6_compare('::ffff:7f00:1', '127.0.0.1') == 0
ipv6_compare('fe80::85d:e82c:9446:7994', 'fe80::85d:e82c:9446:7995')  < 0
ipv6_compare('192.168.1.1/24', '192.168.1.255/24') == 0
ipv6_compare('fe80::85d:e82c:9446:7994/127', 'fe80::85d:e82c:9446:7995/127') == 0
ipv6_compare('fe80::85d:e82c:9446:7994', 'fe80::85d:e82c:9446:7995', 127) == 0
```

**Sintaxis**

`ipv6_compare(`*Expr1* `, ` *Expr2* `[ ,` *PrefixMask*`])`

**Argumentos**

* *Expr1*, *expr2*: expresión de cadena que representa una dirección IPv6 o IPv4. Las cadenas IPv6 e IPv4 se pueden enmascarar mediante la notación de prefijo IP (vea la nota).
* *PrefixMask*: entero de 0 a 128 que representa el número de bits más significativos que se tienen en cuenta.

> [!Note] 
>**Notación de prefijo IP**
> 
>Es habitual definir direcciones IP con `IP-prefix notation` el uso de un carácter de barra diagonal ( `/` ).
>La dirección IP a la izquierda de la barra diagonal ( `/` ) es la dirección IP base y el número (del 1 al 127) a la derecha de la barra diagonal ( `/` ) es el número de 1 bits contiguos de la máscara de la máscara. 
>
> **Ejemplo**: fe80:: 85d: e82c: 9446:7994/120 tendrá un valor de net/SubnetMask asociado que contiene 120 bits contiguos.

**Devuelve**

* `0`: Si la representación larga del primer argumento de cadena IPv6 es igual que el segundo argumento de cadena IPv6.
* `1`: Si la representación larga del primer argumento de cadena IPv6 es mayor que el segundo argumento de cadena IPv6.
* `-1`: Si la representación larga del primer argumento de cadena IPv6 es menor que el segundo argumento de cadena IPv6.
* `null`: Si la conversión de una de las dos cadenas IPv6 no se ha realizado correctamente.

> [!Note]
> La función puede aceptar y comparar los argumentos que representan direcciones de red IPv6 e IPv4. Sin embargo, si el autor de la llamada sabe que los argumentos están en formato IPv4, utilice la función [ipv4_is_compare ()](./ipv4-comparefunction.md) . Esta función dará como resultado un mejor rendimiento en tiempo de ejecución.

## <a name="examples-ipv6ipv4-comparison-equality-cases"></a>Ejemplos: casos de igualdad de comparación de IPv6/IPv4

### <a name="compare-ips-using-the-ip-prefix-notation-specified-inside-the-ipv6ipv4-strings"></a>Comparar direcciones IP mediante la notación de prefijo IP especificada dentro de las cadenas IPv6/IPv4

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(ip1_string:string, ip2_string:string)
[
 // IPv4 are compared as IPv6 addresses
 '192.168.1.1',    '192.168.1.1',       // Equal IPs
 '192.168.1.1/24', '192.168.1.255',     // 24 bit IP4-prefix is used for comparison
 '192.168.1.1',    '192.168.1.255/24',  // 24 bit IP4-prefix is used for comparison
 '192.168.1.1/30', '192.168.1.255/24',  // 24 bit IP4-prefix is used for comparison
  // IPv6 cases
 'fe80::85d:e82c:9446:7994', 'fe80::85d:e82c:9446:7994',         // Equal IPs
 'fe80::85d:e82c:9446:7994/120', 'fe80::85d:e82c:9446:7998',     // 120 bit IP6-prefix is used for comparison
 'fe80::85d:e82c:9446:7994', 'fe80::85d:e82c:9446:7998/120',     // 120 bit IP6-prefix is used for comparison
 'fe80::85d:e82c:9446:7994/120', 'fe80::85d:e82c:9446:7998/120', // 120 bit IP6-prefix is used for comparison
 // Mixed case of IPv4 and IPv6
 '192.168.1.1',      '::ffff:c0a8:0101', // Equal IPs
 '192.168.1.1/24',   '::ffff:c0a8:01ff', // 24 bit IP-prefix is used for comparison
 '::ffff:c0a8:0101', '192.168.1.255/24', // 24 bit IP-prefix is used for comparison
 '::192.168.1.1/30', '192.168.1.255/24', // 24 bit IP-prefix is used for comparison
]
| extend result = ipv6_compare(ip1_string, ip2_string)
```

|ip1_string|ip2_string|result|
|---|---|---|
|192.168.1.1|192.168.1.1|0|
|192.168.1.1/24|192.168.1.255|0|
|192.168.1.1|192.168.1.255/24|0|
|192.168.1.1/30|192.168.1.255/24|0|
|fe80:: 85d: e82c: 9446:7994|fe80:: 85d: e82c: 9446:7994|0|
|fe80:: 85d: e82c: 9446:7994/120|fe80:: 85d: e82c: 9446:7998|0|
|fe80:: 85d: e82c: 9446:7994|fe80:: 85d: e82c: 9446:7998/120|0|
|fe80:: 85d: e82c: 9446:7994/120|fe80:: 85d: e82c: 9446:7998/120|0|
|192.168.1.1|:: ffff: c0a8:0101|0|
|192.168.1.1/24|:: ffff: c0a8:01ff|0|
|:: ffff: c0a8:0101|192.168.1.255/24|0|
|:: 192.168.1.1/30|192.168.1.255/24|0|

### <a name="compare-ips-using-ip-prefix-notation-specified-inside-the-ipv6ipv4-strings-and-as-additional-argument-of-the-ipv6_compare-function"></a>Compara direcciones IP mediante la notación de prefijo IP especificada dentro de las cadenas IPv6/IPv4 y como argumento adicional de la `ipv6_compare()` función.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(ip1_string:string, ip2_string:string, prefix:long)
[
 // IPv4 are compared as IPv6 addresses 
 '192.168.1.1',    '192.168.1.0',   31, // 31 bit IP4-prefix is used for comparison
 '192.168.1.1/24', '192.168.1.255', 31, // 24 bit IP4-prefix is used for comparison
 '192.168.1.1',    '192.168.1.255', 24, // 24 bit IP4-prefix is used for comparison
   // IPv6 cases
 'fe80::85d:e82c:9446:7994', 'fe80::85d:e82c:9446:7995',     127, // 127 bit IP6-prefix is used for comparison
 'fe80::85d:e82c:9446:7994/127', 'fe80::85d:e82c:9446:7998', 120, // 120 bit IP6-prefix is used for comparison
 'fe80::85d:e82c:9446:7994/120', 'fe80::85d:e82c:9446:7998', 127, // 120 bit IP6-prefix is used for comparison
 // Mixed case of IPv4 and IPv6
 '192.168.1.1/24',   '::ffff:c0a8:01ff', 127, // 127 bit IP6-prefix is used for comparison
 '::ffff:c0a8:0101', '192.168.1.255',    120, // 120 bit IP6-prefix is used for comparison
 '::192.168.1.1/30', '192.168.1.255/24', 127, // 120 bit IP6-prefix is used for comparison
]
| extend result = ipv6_compare(ip1_string, ip2_string, prefix)
```

|ip1_string|ip2_string|prefix|result|
|---|---|---|---|
|192.168.1.1|192.168.1.0|31|0|
|192.168.1.1/24|192.168.1.255|31|0|
|192.168.1.1|192.168.1.255|24|0|
|fe80:: 85d: e82c: 9446:7994|fe80:: 85d: e82c: 9446:7995|127|0|
|fe80:: 85d: e82c: 9446:7994/127|fe80:: 85d: e82c: 9446:7998|120|0|
|fe80:: 85d: e82c: 9446:7994/120|fe80:: 85d: e82c: 9446:7998|127|0|
|192.168.1.1/24|:: ffff: c0a8:01ff|127|0|
|:: ffff: c0a8:0101|192.168.1.255|120|0|
|:: 192.168.1.1/30|192.168.1.255/24|127|0|

