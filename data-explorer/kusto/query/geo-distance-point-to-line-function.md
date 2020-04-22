---
title: geo_distance_point_to_line() - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe geo_distance_point_to_line() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/11/2020
ms.openlocfilehash: 34c6811ed5a51dae5281a4374421df5914c5b263
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663783"
---
# <a name="geo_distance_point_to_line"></a>geo_distance_point_to_line()

Calcula la distancia más corta entre una coordenada y una línea en la Tierra.

**Sintaxis**

`geo_distance_point_to_line(`*longitud**latitud*`, `*lineString* `, ``)`

**Argumentos**

* *longitud*: Valor de longitud de coordenadas geoespaciales en grados. El valor válido es un número real y en el intervalo [-180, +180].
* *latitud*: Valor de latitud de coordenadas geoespaciales en grados. El valor válido es un número real y en el rango [-90, +90].
* *lineString*: Line en el [formato GeoJSON](https://tools.ietf.org/html/rfc7946) y de un tipo de datos [dinámico.](./scalar-data-types/dynamic.md)

**Devuelve**

La distancia más corta, en metros, entre una coordenada y una línea en la Tierra. Si la coordenada o lineString no son válidas, la consulta producirá un resultado nulo.

> [!NOTE]
> * Las coordenadas geoespaciales se interpretan como representadas por el sistema de referencia de coordenadas [WGS-84.](https://earth-info.nga.mil/GandG/update/index.php?action=home)
> * El [dato geodésico](https://en.wikipedia.org/wiki/Geodetic_datum) utilizado para medir la distancia en la Tierra es una esfera. Los bordes de línea son geodésicos en la esfera.

**Definición y restricciones de LineString**

dynamic(''type': "LineString","coordinates": [lng_1,lat_1], [lng_2,lat_2] ,..., [lng_N,lat_N] ]).

* LineString matriz de coordenadas debe contener al menos dos entradas.
* Las coordenadas [longitud,latitud] deben ser válidas cuando la longitud es un número real en el rango [-180, +180] y la latitud es un número real en el rango [-90, +90].
* La longitud de la arista debe ser inferior a 180 grados. Se elegirá el borde más corto entre los dos vértices.

> [!TIP]
> Para un mejor rendimiento, utilice líneas literales.

**Ejemplos**

En el ejemplo siguiente se encuentra la distancia más corta entre el aeropuerto de North Las Vegas y una carretera cercana.

:::image type="content" source="images/queries/geo/distance_point_to_line.png" alt-text="Distancia entre el aeropuerto del norte de Las Vegas y la carretera":::

```kusto
print distance_in_meters = geo_distance_point_to_line(-115.199625, 36.210419, dynamic({ "type":"LineString","coordinates":[[-115.115385,36.229195],[-115.136995,36.200366],[-115.140252,36.192470],[-115.143558,36.188523],[-115.144076,36.181954],[-115.154662,36.174483],[-115.166431,36.176388],[-115.183289,36.175007],[-115.192612,36.176736],[-115.202485,36.173439],[-115.225355,36.174365]]}))
```

| distance_in_meters |
|--------------------|
| 3797.88887253334   |

Eventos de tormenta en la costa sur de EE.UU. Los eventos se filtran por una distancia máxima de 5 km de la línea de costa definida.

:::image type="content" source="images/queries/geo/us_south_coast_storm_events.png" alt-text="Eventos de tormenta en la costa sur de EE. UU.":::

```kusto
let southCoast = dynamic({"type":"LineString","coordinates":[[-97.18505859374999,25.997549919572112],[-97.58056640625,26.96124577052697],[-97.119140625,27.955591004642553],[-94.04296874999999,29.726222319395504],[-92.98828125,29.82158272057499],[-89.18701171875,29.11377539511439],[-89.384765625,30.315987718557867],[-87.5830078125,30.221101852485987],[-86.484375,30.4297295750316],[-85.1220703125,29.6880527498568],[-84.00146484374999,30.14512718337613],[-82.6611328125,28.806173508854776],[-82.81494140625,28.033197847676377],[-82.177734375,26.52956523826758],[-80.9912109375,25.20494115356912]]});
StormEvents
| project BeginLon, BeginLat, EventType
| where geo_distance_point_to_line(BeginLon, BeginLat, southCoast) < 5000
| render scatterchart with (kind=map) // map rendering available in Kusto Explorer desktop
```

Recogidas de taxis de NY. Las pastillas se filtran por una distancia máxima de 0,1 m desde la línea definida.

:::image type="content" source="images/queries/geo/park_ave_ny_road.png" alt-text="Eventos de tormenta en la costa sur de EE.UU.":::

```kusto
nyc_taxi
| project pickup_longitude, pickup_latitude
| where geo_distance_point_to_line(pickup_longitude, pickup_latitude, dynamic({"type":"LineString","coordinates":[[-73.97583961486816,40.75486445336327],[-73.95215034484863,40.78743027428638]]})) <= 0.1
| take 50
| render scatterchart with (kind=map) // map rendering available in Kusto Explorer desktop
```

En el ejemplo siguiente se devolverá un resultado nulo debido a la entrada LineString no válida.
```kusto
print distance_in_meters = geo_distance_point_to_line(1,1, dynamic({ "type":"LineString"}))
```

| distance_in_meters |
|--------------------|
|                    |

En el ejemplo siguiente se devolverá un resultado nulo debido a la entrada de coordenadas no válida.
```kusto
print distance_in_meters = geo_distance_point_to_line(300, 3, dynamic({ "type":"LineString","coordinates":[[1,1],[2,2]]}))
```

| distance_in_meters |
|--------------------|
|                    |