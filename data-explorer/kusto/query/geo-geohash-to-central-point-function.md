---
title: 'geo_geohash_to_central_point (): Explorador de datos de Azure'
description: En este artículo se describe geo_geohash_to_central_point () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: mbrichko
ms.service: data-explorer
ms.topic: reference
ms.date: 01/27/2020
ms.openlocfilehash: eb59eae0bc014c6ce9060d65f6c3aced80e4275c
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227135"
---
# <a name="geo_geohash_to_central_point"></a>geo_geohash_to_central_point()

Calcula las coordenadas geoespaciales que representan el centro de un área rectangular de geohash.

Obtenga más información sobre [`geohash`](https://en.wikipedia.org/wiki/Geohash) .  

**Sintaxis**

`geo_geohash_to_central_point(`*geohash*`)`

**Argumentos**

*geohash*: valor de cadena geohash tal como lo calculó [geo_point_to_geohash ()](geo-point-to-geohash-function.md). La cadena geohash puede tener entre 1 y 18 caracteres.

**Devuelve**

Valores de coordenadas geoespaciales en [formato GeoJSON](https://tools.ietf.org/html/rfc7946) y de un tipo de datos [dinámico](./scalar-data-types/dynamic.md) . Si el geohash no es válido, la consulta generará un resultado null.

> [!NOTE]
> El formato GeoJSON especifica la longitud primero y la latitud segundo.

**Ejemplos**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print point = geo_geohash_to_central_point("sunny")
| extend coordinates = point.coordinates
| extend longitude = coordinates[0], latitude = coordinates[1]
```

|point|coordenadas|longitude|latitude|
|---|---|---|---|
|{<br>  "type": "Point",<br>  "coordenadas": [<br>    42.47314453125,<br>    23.70849609375<br>  ]<br>}|[<br>  42.47314453125,<br>  23.70849609375<br>]|42.47314453125|23.70849609375|

En el ejemplo siguiente se devuelve un resultado null debido a la entrada de geohash no válida.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print geohash = geo_geohash_to_central_point("a")
```

|geohash|
|---|
||

## <a name="example-creating-location-deep-links-for-bing-maps"></a>Ejemplo: creación de vínculos profundos de ubicación para Bing Maps

Puede usar el valor de geohash para crear una dirección URL de vínculo profundo a mapas de Bing apuntando al punto del centro de geohash:

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
// Use string concatenation to create Bing Map deep-link URL from a geo-point
let point_to_map_url = (_point:dynamic, _title:string) 
{
    strcat('https://www.bing.com/maps?sp=point.', _point.coordinates[1] ,'_', _point.coordinates[0], '_', url_encode(_title)) 
};
// Convert geohash to center point, and then use 'point_to_map_url' to create Bing Map deep-link
let geohash_to_map_url = (_geohash:string, _title:string)
{
    point_to_map_url(geo_geohash_to_central_point(_geohash), _title)
};
print geohash = 'sv8wzvy7'
| extend url = geohash_to_map_url(geohash, "You are here")
```

|geohash|url|
|---|---|
|sv8wzvy7|[https://www.bing.com/maps?sp=point.32.15620994567871_34.80245590209961_You+are+here](https://www.bing.com/maps?sp=point.32.15620994567871_34.80245590209961_You+are+here)|
