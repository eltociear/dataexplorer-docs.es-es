---
title: ipv4_is_match() - Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe ipv4_is_match() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: aa0646321af2d467c1e4af07fba81ccdc58eff37
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513743"
---
# <a name="ipv4_is_match"></a>ipv4_is_match()

Coincide con dos cadenas IPv4.

```kusto
ipv4_is_match("127.0.0.1", "127.0.0.1") == true
ipv4_is_match('192.168.1.1', '192.168.1.255') == false
ipv4_is_match('192.168.1.1/24', '192.168.1.255/24') == true
ipv4_is_match('192.168.1.1', '192.168.1.255', 24) == true
```

**Sintaxis**

`ipv4_is_match(`*Expr1*`, `*Expr2*`[ ,`*PrefixMask*`])`

**Argumentos**

* *Expr1*, *Expr2*: Una expresión de cadena que representa una dirección IPv4. Las cadenas IPv4 se pueden enmascarar mediante [la notación de prefijo IP.](#ip-prefix-notation)
* *PrefixMask*: Un entero de 0 a 32 que representa el número de bits más significativos que se tienen en cuenta.

### <a name="ip-prefix-notation"></a>Notación de prefijo IP

Es una práctica común definir `IP-prefix notation` direcciones IP`/`usando un carácter de barra diagonal ( ). La dirección IP a la`/`IZQUIERDA de la barra diagonal ( ) es la dirección IP base, y el número (1 a 32) a la derecha de la barra diagonal (`/`) es el número de 1 bits contiguos en la máscara de red. 

Ejemplo: 192.168.2.0/24 tendrá una máscara de red/subred asociada que contenga 24 bits contiguos o 255.255.255.0 en formato decimal punteado.

**Devuelve**

Las dos cadenas IPv4 se analizan y comparan mientras se contabiliza la máscara `PrefixMask` de prefijo IP combinada calculada a partir de prefijos de argumento y el argumento opcional.

Devuelve:
* `true`: si la representación larga del primer argumento de cadena IPv4 es igual al segundo argumento de cadena IPv4.
*  `false`: De lo contrario.

Si la conversión de una de las dos cadenas `null`IPv4 no se realizó correctamente, el resultado será .

**Ejemplos**

## <a name="ipv4-comparison-equality-cases"></a>Casos de igualdad de comparación IPv4

En el ejemplo siguiente se comparan varias direcciones IP mediante la notación de prefijo IP especificada dentro de las cadenas IPv4.

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

En el ejemplo siguiente se comparan varias direcciones IP mediante la notación `ipv4_is_match()` de prefijo IP especificada dentro de las cadenas IPv4 y como argumento adicional de la función.

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