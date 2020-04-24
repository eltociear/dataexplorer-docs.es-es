---
title: extractjson() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe extractjson() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 6177a1c8a6ed4390093e6f6fd24c5f5e9fd04f8a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515341"
---
# <a name="extractjson"></a>extractjson()

Obtenga un elemento especificado fuera de un texto JSON mediante una expresión de ruta. 

Opcionalmente, convierta la cadena extraída en un tipo específico.

```kusto
extractjson("$.hosts[1].AvailableMB", EventText, typeof(int))
```

**Sintaxis**

`extractjson(`*jsonPath* `,` *dataSource*`)` 

**Argumentos**

* *jsonPath*: cadena JsonPath que define un descriptor de acceso en el documento JSON.
* *dataSource*: Un documento JSON.

**Devuelve**

Esta función realiza una consulta de JsonPath en dataSource que contiene una cadena JSON válida, opcionalmente convirtiendo ese valor en otro tipo de acuerdo con el tercer argumento.

**Ejemplo**

La `[``]` notación del corchete`.`y la notación de puntos ( ) son equivalentes:

```kusto
T 
| extend AvailableMB = extractjson("$.hosts[1].AvailableMB", EventText, typeof(int)) 

T
| extend AvailableMD = extractjson("$['hosts'][1]['AvailableMB']", EventText, typeof(int)) 
```

### <a name="json-path-expressions"></a>Expresiones de ruta de JSON

|||
|---|---|
|`$`|Objeto raíz|
|`@`|Objeto actual|
|`.` o `[ ]` | Elemento secundario|
|`[ ]`|Subíndice de matriz|

*(Actualmente no implementamos comodines, recurrencia, unión o segmentos).*


**Consejos de rendimiento**

* Aplique cláusulas where antes de usar `extractjson()`
* En su lugar, considere el uso de una coincidencia de expresión regular con [extract](extractfunction.md) . Esto puede ejecutarse mucho más rápido, y es efectivo si JSON se genera a partir de una plantilla.
* Use `parse_json()` si necesita extraer más de un valor de JSON.
* Considere la posibilidad de analizar el JSON en ingesta declarando que el tipo de columna sea dinámico.