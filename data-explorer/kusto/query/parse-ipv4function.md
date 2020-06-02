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
ms.openlocfilehash: 459b94d4fdb8dbd9d294367b2cee49aab9800406
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84294600"
---
# <a name="parse_ipv4"></a>parse_ipv4()

Convierte la cadena IPv4 en la representación de número larga (con signo 64).

```kusto
parse_ipv4("127.0.0.1") == 2130706433
parse_ipv4('192.1.168.1') < parse_ipv4('192.1.168.2') == true
```

**Sintaxis**

`parse_ipv4(`*`Expr`*`)`

**Argumentos**

* *`Expr`*: Expresión de cadena que representa a IPv4 que se convertirá en Long. La cadena puede incluir la máscara de red mediante [la notación de prefijo IP](#ip-prefix-notation).

## <a name="ip-prefix-notation"></a>Notación de prefijo IP

Las direcciones IP se pueden definir `IP-prefix notation` mediante el uso de un carácter de barra diagonal ( `/` ).
La dirección IP a la izquierda de la barra diagonal ( `/` ) es la dirección IP base. El número (de 1 a 32) a la derecha de la barra diagonal (/) es el número de 1 bit contiguo de la máscara de bits. 

**Ejemplo:** 192.168.2.0/24 tendrá un valor de net/SubnetMask asociado que contiene 24 bits contiguos o 255.255.255.0 en formato decimal con puntos.

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
