---
title: ipv4_compare() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe ipv4_compare() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: ddaaa4e1f9dcf9dfbcd55406cd26e8f1ff4a02fa
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513692"
---
# <a name="ipv4_compare"></a>ipv4_compare()

Compara dos cadenas IPv4.

```kusto
ipv4_compare("127.0.0.1", "127.0.0.1") == 0
ipv4_compare('192.168.1.1', '192.168.1.255') < 0
ipv4_compare('192.168.1.1/24', '192.168.1.255/24') == 0
ipv4_compare('192.168.1.1', '192.168.1.255', 24) == 0
```

**Sintaxis**

`ipv4_compare(`*Expr1*`, `*Expr2*`[ ,`*PrefixMask*`])`

**Argumentos**

* *Expr1*, *Expr2*: Una expresión de cadena que representa una dirección IPv4. Las cadenas IPv4 se pueden enmascarar mediante [la notación de prefijo IP.](#ip-prefix-notation)
* *PrefixMask*: Un entero de 0 a 32 que representa el número de bits más significativos que se tienen en cuenta.

### <a name="ip-prefix-notation"></a>Notación de prefijo IP

Es una práctica común definir `IP-prefix notation` direcciones IP`/`usando un carácter de barra diagonal ( ).
La dirección IP a la`/`IZQUIERDA de la barra diagonal ( ) es la dirección IP base, y el número (1 a 32) a la derecha de la barra diagonal (`/`) es el número de 1 bits contiguos en la máscara de red. 

Ejemplo: 192.168.2.0/24 tendrá una máscara de red/subred asociada que contenga 24 bits contiguos o 255.255.255.0 en formato decimal punteado.

**Devuelve**

Las dos cadenas IPv4 se analizan y comparan mientras se contabiliza la máscara `PrefixMask` de prefijo IP combinada calculada a partir de prefijos de argumento y el argumento opcional.

Devuelve:
* `0`: Si la representación larga del primer argumento de cadena IPv4 es igual al segundo argumento de cadena IPv4
* `1`: Si la representación larga del primer argumento de cadena IPv4 es mayor que el segundo argumento de cadena IPv4
* `-1`: Si la representación larga del primer argumento de cadena IPv4 es menor que el segundo argumento de cadena IPv4

Si la conversión de una de las dos cadenas `null`IPv4 no se realizó correctamente, el resultado será .

## <a name="examples-ipv4-comparison-equality-cases"></a>Ejemplos: Casos de igualdad de comparación IPv4

En el ejemplo siguiente se comparan varias direcciones IP mediante la notación de prefijo IP especificada dentro de las cadenas IPv4.

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

|ip1_string|ip2_string|resultado|
|---|---|---|
|192.168.1.0|192.168.1.0|0|
|192.168.1.1/24|192.168.1.255|0|
|192.168.1.1|192.168.1.255/24|0|
|192.168.1.1/30|192.168.1.255/24|0|

En el ejemplo siguiente se comparan varias direcciones IP mediante la notación `ipv4_compare()` de prefijo IP especificada dentro de las cadenas IPv4 y como argumento adicional de la función.

```kusto
datatable(ip1_string:string, ip2_string:string, prefix:long)
[
 '192.168.1.1',    '192.168.1.0',   31, // 31 bit IP-prefix is used for comparison
 '192.168.1.1/24', '192.168.1.255', 31, // 24 bit IP-prefix is used for comparison
 '192.168.1.1',    '192.168.1.255', 24, // 24 bit IP-prefix is used for comparison
]
| extend result = ipv4_compare(ip1_string, ip2_string, prefix)
```

|ip1_string|ip2_string|prefix|resultado|
|---|---|---|---|
|192.168.1.1|192.168.1.0|31|0|
|192.168.1.1/24|192.168.1.255|31|0|
|192.168.1.1|192.168.1.255|24|0|