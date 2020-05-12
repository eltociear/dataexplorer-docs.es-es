---
title: 'format_bytes (): Explorador de datos de Azure'
description: En este artículo se describe format_bytes () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4da07433be5a052d71740931d4dedd9df0399f56
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227375"
---
# <a name="format_bytes"></a>format_bytes()

Da formato a un número como una cadena que representa el tamaño de los datos en bytes.

```kusto
format_bytes(1024) == '1 KB'"
```

**Sintaxis**

`format_bytes(`*valor* [ `,` *precisión* [ `,` *unidades*]]`)`

**Argumentos**

* `value`: número al que se va a dar formato de tamaño de datos en bytes.
* `precision`: (opcional) número de dígitos al que se redondeará el valor. (el valor predeterminado es 0).
* `units`: (opcional) unidades del tamaño de los datos de destino que utilizará el formato de cadena ( `Bytes` , `KB` , `MB` , `GB` , `TB` , `PB` ). Si el parámetro está vacío, las unidades se seleccionarán automáticamente en función del valor de entrada.

**Devuelve**

Cadena con el resultado de formato.

**Ejemplos**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print 
v1 = format_bytes(564),
v2 = format_bytes(10332, 1),
v3 = format_bytes(20010332),
v4 = format_bytes(20010332, 2),
v5 = format_bytes(20010332, 0, "KB")
```

|v1|v2|v3|v4|V5|
|---|---|---|---|---|
|564 bytes|10,1 KB|19 MB|19,08 MB|19541 KB|
