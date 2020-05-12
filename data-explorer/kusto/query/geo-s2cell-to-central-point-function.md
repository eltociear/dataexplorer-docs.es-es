---
title: 'geo_s2cell_to_central_point (): Explorador de datos de Azure'
description: En este artículo se describe geo_s2cell_to_central_point () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: mbrichko
ms.service: data-explorer
ms.topic: reference
ms.date: 01/27/2020
ms.openlocfilehash: 7eabcb3cb0c3fd001290848e73bb534ff8ea4218
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226829"
---
# <a name="geo_s2cell_to_central_point"></a>geo_s2cell_to_central_point()

Calcula las coordenadas geoespaciales que representan el centro de una celda S2.

Obtenga más información sobre la [jerarquía de celdas S2](https://s2geometry.io/devguide/s2cell_hierarchy).

**Sintaxis**

`geo_s2cell_to_central_point(`*s2cell*`)`

**Argumentos**

*s2cell*: valor de cadena de token de celda S2 tal como lo calculó [geo_point_to_s2cell ()](geo-point-to-s2cell-function.md). La longitud máxima de la cadena del token de celda S2 es de 16 caracteres.

**Devuelve**

Valores de coordenadas geoespaciales en [formato GeoJSON](https://tools.ietf.org/html/rfc7946) y de un tipo de datos [dinámico](./scalar-data-types/dynamic.md) . Si el token de celda S2 no es válido, la consulta generará un resultado null.

> [!NOTE]
> El formato GeoJSON especifica la longitud primero y la latitud segundo.

**Ejemplos**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print point = geo_s2cell_to_central_point("1234567")
| extend coordinates = point.coordinates
| extend longitude = coordinates[0], latitude = coordinates[1]
```

|point|coordenadas|longitude|latitude|
|---|---|---|---|
|{<br>  "type": "Point",<br>  "coordenadas": [<br>    9.86830731850408,<br>    27.468392925827604<br>  ]<br>}|[<br>  9.86830731850408,<br>  27.468392925827604<br>]|9.86830731850408|27.4683929258276|

En el ejemplo siguiente se devuelve un resultado null debido a la entrada del token de celda S2 no válida.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print point = geo_s2cell_to_central_point("a")
```

|point|
|---|
||
