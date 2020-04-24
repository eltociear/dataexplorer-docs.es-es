---
title: geo_s2cell_to_central_point() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe geo_s2cell_to_central_point() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/27/2020
ms.openlocfilehash: 624d163b508768d0a5316ee3ec62a12b11942176
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514440"
---
# <a name="geo_s2cell_to_central_point"></a>geo_s2cell_to_central_point()

Calcula las coordenadas geoespaciales que representan el centro de la celda S2.

Para obtener más información sobre las celdas S2, haga clic [aquí](http://s2geometry.io/devguide/s2cell_hierarchy).

**Sintaxis**

`geo_s2cell_to_central_point(`*s2cell*`)`

**Argumentos**

*s2cell*: S2 Cell Token string value as was calculated by [geo_point_to_s2cell()](geo-point-to-s2cell-function.md). La longitud máxima de cadena del token de celda S2 es de 16 caracteres.

**Devuelve**

Los valores de coordenadas geoespaciales en [formato GeoJSON](https://tools.ietf.org/html/rfc7946) y de un tipo de datos [dinámico.](./scalar-data-types/dynamic.md) Si el token de celda S2 no es válido, la consulta producirá un resultado nulo.

> [!NOTE]
> El formato GeoJSON especifica la longitud primero y la latitud en segundo lugar.

**Ejemplos**

```kusto
print point = geo_s2cell_to_central_point("1234567")
| extend coordinates = point.coordinates
| extend longitude = coordinates[0], latitude = coordinates[1]
```

|point|coordenadas|longitude|latitude|
|---|---|---|---|
|{<br>  "type": "Punto",<br>  "coordenadas": [<br>    9.86830731850408,<br>    27.468392925827604<br>  ]<br>}|[<br>  9.86830731850408,<br>  27.468392925827604<br>]|9.86830731850408|27.4683929258276|

En el ejemplo siguiente se devuelve un resultado nulo debido a la entrada de token de celda S2 no válida.
```kusto
print point = geo_s2cell_to_central_point("a")
```

|point|
|---|
||