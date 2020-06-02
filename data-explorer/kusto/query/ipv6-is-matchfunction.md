---
title: 'ipv6_is_match (): Explorador de datos de Azure'
description: En este artículo se describe la función ipv6_is_match () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: alexans
ms.service: data-explorer
ms.topic: reference
ms.date: 05/27/2020
ms.openlocfilehash: d5bd270e016f1694c28f663a7fe8bf1d9a8f0903
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84301289"
---
# <a name="ipv6_is_match"></a>ipv6_is_match ()

Coincide con dos cadenas de dirección de red IPv6 o IPv4. Las dos cadenas IPv6/IPv4 se analizan y se comparan mientras se cuenta la máscara de prefijo de IP combinada calculada a partir de los prefijos de argumento y el `PrefixMask` argumento opcional.

```kusto
ipv6_is_match('::ffff:7f00:1', '127.0.0.1') == true
ipv6_is_match('fe80::85d:e82c:9446:7994', 'fe80::85d:e82c:9446:7995') == false
ipv6_is_match('192.168.1.1/24', '192.168.1.255/24') == true
ipv6_is_match('fe80::85d:e82c:9446:7994/127', 'fe80::85d:e82c:9446:7995/127') == true
ipv6_is_match('fe80::85d:e82c:9446:7994', 'fe80::85d:e82c:9446:7995', 127) == true
```

**Sintaxis**

`ipv6_is_match(`*Expr1* `, ` *Expr2* `[ ,` *PrefixMask*`])`

**Argumentos**

* *Expr1*, *expr2*: expresión de cadena que representa una dirección IPv6 o IPv4. Las cadenas IPv6 e IPv4 se pueden enmascarar mediante [la notación de prefijo IP](#ip-prefix-notation).
* *PrefixMask*: entero de 0 a 128 que representa el número de bits más significativos que se tienen en cuenta.

## <a name="ip-prefix-notation"></a>Notación de prefijo IP
 
Las direcciones IP se pueden definir `IP-prefix notation` mediante el uso de un carácter de barra diagonal ( `/` ).
La dirección IP a la izquierda de la barra diagonal ( `/` ) es la dirección IP base. El número (1 a 127) a la derecha de la barra diagonal ( `/` ) es el número de 1 bit contiguo de la máscara de bits. 

**Ejemplo**: fe80:: 85d: e82c: 9446:7994/120 tendrá un valor de net/SubnetMask asociado que contiene 120 bits contiguos.

**Devuelve**

* `true`: Si la representación larga del primer argumento de cadena IPv6/IPv4 es igual que el segundo argumento de cadena IPv6/IPv4.
* `false`Casos.
* `null`: Si la conversión de una de las dos cadenas IPv6/IPv4 no se ha realizado correctamente.

> [!Note]
> La función puede aceptar y comparar los argumentos que representan direcciones de red IPv6 e IPv4. Si el autor de la llamada sabe que los argumentos están en formato IPv4, utilice la función [ipv4_is_match ()](./ipv4-is-matchfunction.md) . Esta función dará como resultado un mejor rendimiento en tiempo de ejecución.

## <a name="examples"></a>Ejemplos

### <a name="ipv6ipv4-comparison-equality-case---ip-prefix-notation-specified-inside-the-ipv6ipv4-strings"></a>Caso de igualdad de comparación IPv6/IPv4: notación de prefijo IP especificada dentro de las cadenas IPv6/IPv4

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
| extend result = ipv6_is_match(ip1_string, ip2_string)
```

|ip1_string|ip2_string|resultado|
|---|---|---|
|192.168.1.1|192.168.1.1|1|
|192.168.1.1/24|192.168.1.255|1|
|192.168.1.1|192.168.1.255/24|1|
|192.168.1.1/30|192.168.1.255/24|1|
|fe80:: 85d: e82c: 9446:7994|fe80:: 85d: e82c: 9446:7994|1|
|fe80:: 85d: e82c: 9446:7994/120|fe80:: 85d: e82c: 9446:7998|1|
|fe80:: 85d: e82c: 9446:7994|fe80:: 85d: e82c: 9446:7998/120|1|
|fe80:: 85d: e82c: 9446:7994/120|fe80:: 85d: e82c: 9446:7998/120|1|
|192.168.1.1|:: ffff: c0a8:0101|1|
|192.168.1.1/24|:: ffff: c0a8:01ff|1|
|:: ffff: c0a8:0101|192.168.1.255/24|1|
|:: 192.168.1.1/30|192.168.1.255/24|1|


### <a name="ipv6ipv4-comparison-equality-case--ip-prefix-notation-specified-inside-the-ipv6ipv4-strings-and-as-additional-argument-of-the-ipv6_is_match-function"></a>Mayúsculas/minúsculas de comparación de IPv6/IPv4: notación de prefijo IP especificada dentro de las cadenas IPv6/IPv4 y como argumento adicional de la `ipv6_is_match()` función

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
| extend result = ipv6_is_match(ip1_string, ip2_string, prefix)
```

|ip1_string|ip2_string|prefix|resultado|
|---|---|---|---|
|192.168.1.1|192.168.1.0|31|1|
|192.168.1.1/24|192.168.1.255|31|1|
|192.168.1.1|192.168.1.255|24|1|
|fe80:: 85d: e82c: 9446:7994|fe80:: 85d: e82c: 9446:7995|127|1|
|fe80:: 85d: e82c: 9446:7994/127|fe80:: 85d: e82c: 9446:7998|120|1|
|fe80:: 85d: e82c: 9446:7994/120|fe80:: 85d: e82c: 9446:7998|127|1|
|192.168.1.1/24|:: ffff: c0a8:01ff|127|1|
|:: ffff: c0a8:0101|192.168.1.255|120|1|
|:: 192.168.1.1/30|192.168.1.255/24|127|1|
