---
title: geo_distance_2points() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe geo_distance_2points() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/11/2020
ms.openlocfilehash: 30a26bbcc57ecde2ac3beeeb1483b953a0cc5820
ms.sourcegitcommit: 29018b3db4ea7d015b1afa65d49ecf918cdff3d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "82030175"
---
# <a name="geo_distance_2points"></a>geo_distance_2points()

Calcula la distancia más corta entre dos coordenadas geoespaciales en la Tierra.

**Sintaxis**

`geo_distance_2points(`*p2_latitude*`, `*p2_longitude**p2_latitude* `, `*p1_latitude*`, `p1_longitude p1_latitude`)`

**Argumentos**

* *p1_longitude*: Primera coordenada geoespacial, valor de longitud en grados. El valor válido es un número real y en el intervalo [-180, +180].
* *p1_latitude*: Primera coordenada geoespacial, valor de latitud en grados. El valor válido es un número real y en el rango [-90, +90].
* *p2_longitude*: Segunda coordenada geoespacial, valor de longitud en grados. El valor válido es un número real y en el intervalo [-180, +180].
* *p2_latitude*: Segunda coordenada geoespacial, valor de latitud en grados. El valor válido es un número real y en el rango [-90, +90].

**Devuelve**

La distancia más corta, en metros, entre dos ubicaciones geográficas en la Tierra. Si las coordenadas no son válidas, la consulta producirá un resultado nulo.

> [!NOTE]
> * Las coordenadas geoespaciales se interpretan como representadas por el sistema de referencia de coordenadas [WGS-84.](https://earth-info.nga.mil/GandG/update/index.php?action=home)
> * El [dato geodésico](https://en.wikipedia.org/wiki/Geodetic_datum) utilizado para medir la distancia en la Tierra es una esfera.

**Ejemplos**

En el ejemplo siguiente se encuentra la distancia más corta entre Seattle y Los Angeles.


:::image type="content" source="images/geo-distance-2points-function/distance_2points_seattle_los_angeles.png" alt-text="Distancia entre Seattle y Los Angeles":::

```kusto
print distance_in_meters = geo_distance_2points(-122.407628, 47.578557, -118.275287, 34.019056)
```

| distance_in_meters |
|--------------------|
| 1546754.35197381   |

Aquí hay una aproximación del camino más corto de Seattle a Londres. La línea consta de coordenadas a lo largo de LineString y a menos de 500 metros de ella.

:::image type="content" source="images/geo-distance-2points-function/line_seattle_london.png" alt-text="De Seattle a London LineString":::

```kusto
range i from 1 to 1000000 step 1
| project lng = rand() * real(-122), lat = rand() * 90
| where lng between(real(-122) .. 0) and lat between(47 .. 90)
| where geo_distance_point_to_line(lng,lat,dynamic({"type":"LineString","coordinates":[[-122,47],[0,51]]})) < 500
| render scatterchart with (kind=map) // map rendering available in Kusto Explorer desktop
```

En el ejemplo siguiente se encuentran todas las filas en las que la distancia más corta entre dos coordenadas está entre 1 y 11 metros.
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

En el ejemplo siguiente se devuelve un resultado nulo debido a la entrada de coordenadas no válida.
```kusto
print distance = geo_distance_2points(300,1,1,1)
```

| distancia |
|----------|
|          |