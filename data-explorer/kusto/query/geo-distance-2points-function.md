---
title: 'geo_distance_2points (): Explorador de datos de Azure'
description: En este artículo se describe geo_distance_2points () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: mbrichko
ms.service: data-explorer
ms.topic: reference
ms.date: 03/11/2020
ms.openlocfilehash: 64c61bcde7bfe02d55bb0f4a6719e9d7b1c1a681
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227424"
---
# <a name="geo_distance_2points"></a>geo_distance_2points()

Calcula la distancia más corta entre dos coordenadas geoespaciales de la tierra.

**Sintaxis**

`geo_distance_2points(`*p1_longitude* `, ` *p1_latitude* `, ` *p2_longitude* `, ` *p2_latitude*`)`

**Argumentos**

* *p1_longitude*: primera coordenada geoespacial, valor de longitud en grados. El valor válido es un número real y en el intervalo [-180, + 180].
* *p1_latitude*: primera coordenada geoespacial, valor de latitud en grados. El valor válido es un número real y en el intervalo [-90, + 90].
* *p2_longitude*: segunda coordenada geoespacial, valor de longitud en grados. El valor válido es un número real y en el intervalo [-180, + 180].
* *p2_latitude*: segunda coordenada geoespacial, valor de latitud en grados. El valor válido es un número real y en el intervalo [-90, + 90].

**Devuelve**

Distancia más corta, en metros, entre dos ubicaciones geográficas de la tierra. Si las coordenadas no son válidas, la consulta generará un resultado null.

> [!NOTE]
> * Las coordenadas geoespaciales se interpretan como representadas por el sistema [de referencia de coordenadas WGS-84](https://earth-info.nga.mil/GandG/update/index.php?action=home) .
> * El [geodésico de referencia](https://en.wikipedia.org/wiki/Geodetic_datum) que se utiliza para medir la distancia en la tierra es una esfera.

**Ejemplos**

En el ejemplo siguiente se busca la distancia más corta entre Seattle y los Ángeles.

:::image type="content" source="images/geo-distance-2points-function/distance_2points_seattle_los_angeles.png" alt-text="Distancia entre Seattle y los Ángeles":::

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print distance_in_meters = geo_distance_2points(-122.407628, 47.578557, -118.275287, 34.019056)
```

| distance_in_meters |
|--------------------|
| 1546754,35197381   |

Esta es una aproximación de la ruta más corta de Seattle a Londres. La línea consta de coordenadas a lo largo del LineString y dentro de 500 metros de ella.

:::image type="content" source="images/geo-distance-2points-function/line_seattle_london.png" alt-text="Seattle a Londres LineString":::

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
range i from 1 to 1000000 step 1
| project lng = rand() * real(-122), lat = rand() * 90
| where lng between(real(-122) .. 0) and lat between(47 .. 90)
| where geo_distance_point_to_line(lng,lat,dynamic({"type":"LineString","coordinates":[[-122,47],[0,51]]})) < 500
| render scatterchart with (kind=map) // map rendering available in Kusto Explorer desktop
```

En el siguiente ejemplo se buscan todas las filas en las que la distancia más corta entre dos coordenadas se encuentra entre 1 y 11 metros.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
StormEvents
| extend distance_1_to_11m = geo_distance_2points(BeginLon, BeginLat, EndLon, EndLat)
| where distance_1_to_11m between (1 .. 11)
| project distance_1_to_11m
```

| distance_1_to_11m |
|-------------------|
| 10.5723100154958  |
| 7.92153588248414  |

En el ejemplo siguiente se devuelve un resultado null debido a la entrada de coordenadas no válida.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print distance = geo_distance_2points(300,1,1,1)
```

| distancia |
|----------|
|          |
