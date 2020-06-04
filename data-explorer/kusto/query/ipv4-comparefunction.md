---
title: 'ipv4_compare (): Explorador de datos de Azure'
description: En este artículo se describe ipv4_compare () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 67887aac4ab04e016ed63045e66ebcfab343c135
ms.sourcegitcommit: 8953d09101f4358355df60ab09e55e71bc255ead
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2020
ms.locfileid: "84420873"
---
# <a name="ipv4_compare"></a>ipv4_compare()

Compara dos cadenas IPv4. Las dos cadenas IPv4 se analizan y se comparan mientras se cuenta la máscara de prefijo de IP combinada calculada a partir de los prefijos de argumento y el `PrefixMask` argumento opcional.

```kusto
ipv4_compare("127.0.0.1", "127.0.0.1") == 0
ipv4_compare('192.168.1.1', '192.168.1.255') < 0
ipv4_compare('192.168.1.1/24', '192.168.1.255/24') == 0
ipv4_compare('192.168.1.1', '192.168.1.255', 24) == 0
```

**Sintaxis**

`ipv4_compare(`*Expr1* `, ` *Expr2* `[ ,` *PrefixMask*`])`

**Argumentos**

* *Expr1*, *expr2*: expresión de cadena que representa una dirección IPv4. Las cadenas IPv4 se pueden enmascarar mediante [la notación de prefijo IP](#ip-prefix-notation).
* *PrefixMask*: entero de 0 a 32 que representa el número de bits más significativos que se tienen en cuenta.

## <a name="ip-prefix-notation"></a>Notación de prefijo IP
 
Las direcciones IP se pueden definir `IP-prefix notation` mediante el uso de un carácter de barra diagonal ( `/` ).
La dirección IP a la izquierda de la barra diagonal ( `/` ) es la dirección IP base. El número (1 a 32) a la derecha de la barra diagonal ( `/` ) es el número de 1 bit contiguo de la máscara de bits. 

**Ejemplo:** 192.168.2.0/24 tendrá un valor de net/SubnetMask asociado que contiene 24 bits contiguos o 255.255.255.0 en formato decimal con puntos.

**Devuelve**

* `0`: Si la representación larga del primer argumento de cadena IPv4 es igual que el segundo argumento de cadena IPv4
* `1`: Si la representación larga del primer argumento de cadena IPv4 es mayor que el segundo argumento de cadena IPv4
* `-1`: Si la representación larga del primer argumento de cadena IPv4 es menor que el segundo argumento de cadena IPv4
* `null`: Si la conversión de una de las dos cadenas IPv4 no se ha realizado correctamente.

## <a name="examples-ipv4-comparison-equality-cases"></a>Ejemplos: casos de igualdad de comparación de IPv4

### <a name="compare-ips-using-the-ip-prefix-notation-specified-inside-the-ipv4-strings"></a>Comparar direcciones IP mediante la notación de prefijo IP especificada dentro de las cadenas de IPv4

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(ip1_string:string, ip2_string:string)
[
 '192.168.1.0',    '192.168.1.0',       // Equal IPs
 '192.168.1.1/24', '192.168.1.255',     // 24 bit IP-prefix is used for comparison
 '192.168.1.1',    '192.168.1.255/24',  // 24 bit IP-prefix is used for comparison
 '192.168.1.1/30', '192.168.1.255/24',  // 24 bit IP-prefix is used for comparison
]
| extend result = ipv4_compare(ip1_string, ip2_string)
```

|ip1_string|ip2_string|result|
|---|---|---|
|192.168.1.0|192.168.1.0|0|
|192.168.1.1/24|192.168.1.255|0|
|192.168.1.1|192.168.1.255/24|0|
|192.168.1.1/30|192.168.1.255/24|0|

### <a name="compare-ips-using-ip-prefix-notation-specified-inside-the-ipv4-strings-and-as-additional-argument-of-the-ipv4_compare-function"></a>Compara direcciones IP mediante la notación de prefijo IP especificada dentro de las cadenas IPv4 y como argumento adicional de la `ipv4_compare()` función.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(ip1_string:string, ip2_string:string, prefix:long)
[
 '192.168.1.1',    '192.168.1.0',   31, // 31 bit IP-prefix is used for comparison
 '192.168.1.1/24', '192.168.1.255', 31, // 24 bit IP-prefix is used for comparison
 '192.168.1.1',    '192.168.1.255', 24, // 24 bit IP-prefix is used for comparison
]
| extend result = ipv4_compare(ip1_string, ip2_string, prefix)
```

|ip1_string|ip2_string|prefix|result|
|---|---|---|---|
|192.168.1.1|192.168.1.0|31|0|
|192.168.1.1/24|192.168.1.255|31|0|
|192.168.1.1|192.168.1.255|24|0|

