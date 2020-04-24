---
title: format_bytes() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe format_bytes() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 5d5bfa234e001f498737b9df88274096f8592f52
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515171"
---
# <a name="format_bytes"></a>format_bytes()

Da formato a un número como una cadena que representa el tamaño de los datos en bytes.

```kusto
format_bytes(1024) == '1 KB'"
```

**Sintaxis**

`format_bytes(`*valor* `,` *precision* [`,` precisión [ *unidades*]]`)`

**Argumentos**

* `value`: un número que se formateará como tamaño de datos en bytes.
* `precision`: (opcional) Número de dígitos a los que se redondeará el valor. (el valor predeterminado es 0).
* `units`: (opcional) Unidades del tamaño de datos`Bytes` `KB`de `MB` `GB`destino `TB`que utilizará el formato de cadena ( , , , , , ). `PB` Si el parámetro está vacío- las unidades se seleccionarán automáticamente en función del valor de entrada.

**Devuelve**

Cadena con el resultado del formato.

**Ejemplos**

```kusto
print 
v1 = format_bytes(564),
v2 = format_bytes(10332, 1),
v3 = format_bytes(20010332),
v4 = format_bytes(20010332, 2),
v5 = format_bytes(20010332, 0, "KB")
```

|v1|v2|v3|v4|v5|
|---|---|---|---|---|
|564 Bytes|10.1 KB|19 MB|19.08 MB|19541 KB|