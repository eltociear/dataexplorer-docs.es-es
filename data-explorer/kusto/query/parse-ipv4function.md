---
title: 'parse_ipv4 (): Explorador de datos de Azure'
description: En este artículo se describe parse_ipv4 () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 2bfea891b6057b5d43b65fa045e2b01d5e025a82
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271254"
---
# <a name="parse_ipv4"></a>parse_ipv4()

Convierte la cadena IPv4 en la representación de número larga (con signo 64).

```kusto
parse_ipv4("127.0.0.1") == 2130706433
parse_ipv4('192.1.168.1') < parse_ipv4('192.1.168.2') == true
```

**Sintaxis**

`parse_ipv4(`*Argumento*`)`

**Argumentos**

* *Expr*: expresión de cadena que representa IPv4 que se convertirá en Long. La cadena puede incluir la máscara de red mediante [la notación de prefijo IP](#ip-prefix-notation).

### <a name="ip-prefix-notation"></a>Notación de prefijo IP

Es una práctica común definir direcciones IP mediante `IP-prefix notation` el uso de un carácter de barra diagonal ( `/` ).
La dirección IP a la izquierda de la barra diagonal ( `/` ) es la dirección IP base y el número (1 a 32) a la derecha de la barra diagonal (/) es el número de 1 bits contiguos de la máscara de la máscara. Por lo tanto, 192.168.2.0/24 tendrá un valor de net/SubnetMask asociado que contiene 24 bits contiguos o 255.255.255.0 en formato decimal con puntos.

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un número largo.
Si la conversión no se realiza correctamente, el resultado será `null` .
 
**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
datatable(ip_string:string)
[
 '192.168.1.1',
 '192.168.1.1/24',
 '255.255.255.255/31'
]
| extend ip_long = parse_ipv4(ip_string)
```

|ip_string|ip_long|
|---|---|
|192.168.1.1|3232235777|
|192.168.1.1/24|3232235776|
|255.255.255.255/31|4294967294|
