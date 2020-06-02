---
title: 'ipv4_is_match (): Explorador de datos de Azure'
description: En este artículo se describe ipv4_is_match () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 7b076f9878828ca8503c808b6ab94daf3375e2d4
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84294634"
---
# <a name="ipv4_is_match"></a>ipv4_is_match ()

Coincide con dos cadenas IPv4. Las dos cadenas IPv4 se analizan y se comparan mientras se cuenta la máscara de prefijo de IP combinada calculada a partir de los prefijos de argumento y el `PrefixMask` argumento opcional.

```kusto
ipv4_is_match("127.0.0.1", "127.0.0.1") == true
ipv4_is_match('192.168.1.1', '192.168.1.255') == false
ipv4_is_match('192.168.1.1/24', '192.168.1.255/24') == true
ipv4_is_match('192.168.1.1', '192.168.1.255', 24) == true
```

**Sintaxis**

`ipv4_is_match(`*Expr1* `, ` *Expr2* `[ ,` *PrefixMask*`])`

**Argumentos**

* *Expr1*, *expr2*: expresión de cadena que representa una dirección IPv4. Las cadenas IPv4 se pueden enmascarar mediante [la notación de prefijo IP](#ip-prefix-notation).
* *PrefixMask*: entero de 0 a 32 que representa el número de bits más significativos que se tienen en cuenta.

## <a name="ip-prefix-notation"></a>Notación de prefijo IP

Las direcciones IP se pueden definir `IP-prefix notation` mediante el uso de un carácter de barra diagonal ( `/` ). La dirección IP a la izquierda de la barra diagonal ( `/` ) es la dirección IP base. El número (1 a 32) a la derecha de la barra diagonal ( `/` ) es el número de 1 bit contiguo de la máscara de bits. 
**Ejemplo:** 192.168.2.0/24 tendrá un valor de net/SubnetMask asociado que contiene 24 bits contiguos o 255.255.255.0 en formato decimal con puntos.

**Devuelve**

* `true`: Si la representación larga del primer argumento de cadena IPv4 es igual que el segundo argumento de cadena IPv4.
*  `false`Casos.
* `null`: Si la conversión de una de las dos cadenas IPv4 no se ha realizado correctamente.

## <a name="examples"></a>Ejemplos

### <a name="ipv4-comparison-equality---ip-prefix-notation-specified-inside-the-ipv4-strings"></a>Igualdad de comparación IPv4: notación de prefijo IP especificada dentro de las cadenas IPv4.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(ip1_string:string, ip2_string:string)
[
 '192.168.1.0',    '192.168.1.0',       // Equal IPs
 '192.168.1.1/24', '192.168.1.255',     // 24 bit IP-prefix is used for comparison
 '192.168.1.1',    '192.168.1.255/24',  // 24 bit IP-prefix is used for comparison
 '192.168.1.1/30', '192.168.1.255/24',  // 24 bit IP-prefix is used for comparison
]
| extend result = ipv4_is_match(ip1_string, ip2_string)
```

|ip1_string|ip2_string|resultado|
|---|---|---|
|192.168.1.0|192.168.1.0|1|
|192.168.1.1/24|192.168.1.255|1|
|192.168.1.1|192.168.1.255/24|1|
|192.168.1.1/30|192.168.1.255/24|1|

### <a name="ipv4-comparison-equality---ip-prefix-notation-specified-inside-the-ipv4-strings-and-an-additional-argument-of-the-ipv4_is_match-function"></a>Igualdad de comparación IPv4: notación de prefijo IP especificada dentro de las cadenas IPv4 y un argumento adicional de la `ipv4_is_match()` función

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(ip1_string:string, ip2_string:string, prefix:long)
[
 '192.168.1.1',    '192.168.1.0',   31, // 31 bit IP-prefix is used for comparison
 '192.168.1.1/24', '192.168.1.255', 31, // 24 bit IP-prefix is used for comparison
 '192.168.1.1',    '192.168.1.255', 24, // 24 bit IP-prefix is used for comparison
]
| extend result = ipv4_is_match(ip1_string, ip2_string, prefix)
```

|ip1_string|ip2_string|prefix|resultado|
|---|---|---|---|
|192.168.1.1|192.168.1.0|31|1|
|192.168.1.1/24|192.168.1.255|31|1|
|192.168.1.1|192.168.1.255|24|1|
