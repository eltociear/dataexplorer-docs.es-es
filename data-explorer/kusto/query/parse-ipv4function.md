---
title: parse_ipv4() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe parse_ipv4() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 296c7e77a2397501ae086e16154ca249249c23e4
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511754"
---
# <a name="parse_ipv4"></a>parse_ipv4()

Convierte la cadena IPv4 en una representación numérica larga (firmada de 64 bits).

```kusto
parse_ipv4("127.0.0.1") == 2130706433
parse_ipv4('192.1.168.1') < parse_ipv4('192.1.168.2') == true
```

**Sintaxis**

`parse_ipv4(`*Expr*`)`

**Argumentos**

* *Expr*: Expresión de cadena que representa IPv4 que se convertirá a long. String puede incluir net-mask mediante [la notación de prefijo IP.](#ip-prefix-notation)

### <a name="ip-prefix-notation"></a>Notación de prefijo IP

Es una práctica común definir `IP-prefix notation` direcciones IP`/`usando un carácter de barra diagonal ( ).
La dirección IP a la`/`IZQUIERDA de la barra diagonal ( ) es la dirección IP base, y el número (1 a 32) a la derecha de la barra diagonal (/) es el número de 1 bits contiguos en la máscara de red. Por lo tanto, 192.168.2.0/24 tendrá una máscara de red/subred asociada que contiene 24 bits contiguos o 255.255.255.0 en formato decimal punteado.

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un número largo.
Si la conversión no `null`se realiza correctamente, el resultado será .
 
**Ejemplo**

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