---
title: parse_version() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe parse_version() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 5cb35c12849568f24a6bde42461e8af66058f48f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511720"
---
# <a name="parse_version"></a>parse_version()

Convierte la representación de cadena de entrada de la versión en un número decimal comparable.

```kusto
parse_version("0.0.0.1")
```

**Sintaxis**

`parse_version``(` *Expr*`)`

**Argumentos**

* *Expr*: Una expresión escalar `string` de tipo que especifica la versión que se va a analizar.

**Devuelve**

Si la conversión se realiza correctamente, el resultado será un decimal.
Si la conversión no `null`se realiza correctamente, el resultado será .

**Notas**

La cadena de entrada debe contener de 1 a 4 partes de la versión, representadas como números y separadas por puntos ('.').

Cada parte de la versión puede contener hasta 8 dígitos (valor máximo - 99999999).

En caso de que la cantidad de piezas sea inferior a`1.0` == `1.0.0.0`4, todas las piezas faltantes se consideran como finales ( ).

 
**Ejemplo**
```kusto
let dt = datatable(v:string)
["0.0.0.5","0.0.7.0","0.0.3","0.2","0.1.2.0","1.2.3.4","1","99999999.0.0.0"];
dt | project v1=v, _key=1 
| join kind=inner (dt | project v2=v, _key = 1) on _key | where v1 != v2
| summarize v1 = max(v1),v2 = min(v2) by (hash(v1) + hash(v2)) // removing duplications
| project v1, v2, higher_version = iif(parse_version(v1) > parse_version(v2), v1, v2)

```

|v1|v2|higher_version|
|---|---|---|
|99999999.0.0.0|0.0.0.5|99999999.0.0.0|
|1|0.0.0.5|1|
|1.2.3.4|0.0.0.5|1.2.3.4|
|0.1.2.0|0.0.0.5|0.1.2.0|
|0,2|0.0.0.5|0,2|
|0.0.3|0.0.0.5|0.0.3|
|0.0.7.0|0.0.0.5|0.0.7.0|
|99999999.0.0.0|0.0.7.0|99999999.0.0.0|
|1|0.0.7.0|1|
|1.2.3.4|0.0.7.0|1.2.3.4|
|0.1.2.0|0.0.7.0|0.1.2.0|
|0,2|0.0.7.0|0,2|
|0.0.7.0|0.0.3|0.0.7.0|
|99999999.0.0.0|0.0.3|99999999.0.0.0|
|1|0.0.3|1|
|1.2.3.4|0.0.3|1.2.3.4|
|0.1.2.0|0.0.3|0.1.2.0|
|0,2|0.0.3|0,2|
|99999999.0.0.0|0,2|99999999.0.0.0|
|1|0,2|1|
|1.2.3.4|0,2|1.2.3.4|
|0,2|0.1.2.0|0,2|
|99999999.0.0.0|0.1.2.0|99999999.0.0.0|
|1|0.1.2.0|1|
|1.2.3.4|0.1.2.0|1.2.3.4|
|99999999.0.0.0|1.2.3.4|99999999.0.0.0|
|1.2.3.4|1|1.2.3.4|
|99999999.0.0.0|1|99999999.0.0.0|




