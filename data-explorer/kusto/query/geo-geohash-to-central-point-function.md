---
title: geo_geohash_to_central_point() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe geo_geohash_to_central_point() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/27/2020
ms.openlocfilehash: a71265b58cffcec2fcf9d6ac8d412c8dd44866f4
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81514644"
---
# <a name="geo_geohash_to_central_point"></a>geo_geohash_to_central_point()

Calcula las coordenadas geoespaciales que representan el centro de un área rectangular de Geohash.

Para obtener más información sobre Geohash, consulte [Wikipedia](https://en.wikipedia.org/wiki/Geohash).  

**Sintaxis**

`geo_geohash_to_central_point(`*geohash*`)`

**Argumentos**

*geohash*: Valor de la cadena Geohash tal como fue calculado por [geo_point_to_geohash()](geo-point-to-geohash-function.md). La cadena de geohash puede tener de 1 a 18 caracteres.

**Devuelve**

Los valores de coordenadas geoespaciales en [formato GeoJSON](https://tools.ietf.org/html/rfc7946) y de un tipo de datos [dinámico.](./scalar-data-types/dynamic.md) Si el geohash no es válido, la consulta producirá un resultado nulo.

> [!NOTE]
> El formato GeoJSON especifica la longitud primero y la latitud en segundo lugar.

**Ejemplos**

```kusto
print point = geo_geohash_to_central_point("sunny")
| extend coordinates = point.coordinates
| extend longitude = coordinates[0], latitude = coordinates[1]
```

|point|coordenadas|longitude|latitude|
|---|---|---|---|
|{<br>  "type": "Punto",<br>  "coordenadas": [<br>    42.47314453125,<br>    23.70849609375<br>  ]<br>}|[<br>  42.47314453125,<br>  23.70849609375<br>]|42.47314453125|23.70849609375|

En el ejemplo siguiente se devuelve un resultado nulo debido a la entrada geohash no válida.

```kusto
print geohash = geo_geohash_to_central_point("a")
```

|geohash|
|---|
||

## <a name="example-creating-location-deep-links-for-bing-maps"></a>Ejemplo: Creación de vínculos profundos de ubicación para Bing Maps

Puede usar el valor Geohash para crear una dirección URL de vínculo profundo a Mapas de Bing apuntando al punto central de Geohash:

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