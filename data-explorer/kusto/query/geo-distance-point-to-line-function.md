---
title: 'geo_distance_point_to_line (): Explorador de datos de Azure'
description: En este artículo se describe geo_distance_point_to_line () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: mbrichko
ms.service: data-explorer
ms.topic: reference
ms.date: 03/11/2020
ms.openlocfilehash: 304b40a00fd471b7735ff11c01bdaa8b6ca3a8ec
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227483"
---
# <a name="geo_distance_point_to_line"></a>geo_distance_point_to_line()

Calcula la distancia más corta entre una coordenada y una línea de tierra.

**Sintaxis**

`geo_distance_point_to_line(`*longitud* `, ` *latitud* `, ` *lineString*`)`

**Argumentos**

* *longitud*: valor de longitud de coordenadas geoespaciales en grados. El valor válido es un número real y en el intervalo [-180, + 180].
* *latitud*: valor de latitud de coordenadas geoespaciales en grados. El valor válido es un número real y en el intervalo [-90, + 90].
* *lineString*: línea en el [formato GeoJSON](https://tools.ietf.org/html/rfc7946) y de un tipo de datos [dinámico](./scalar-data-types/dynamic.md) .

**Devuelve**

Distancia más corta, en metros, entre una coordenada y una línea de la tierra. Si las coordenadas o lineString no son válidas, la consulta generará un resultado null.

> [!NOTE]
> * Las coordenadas geoespaciales se interpretan como representadas por el sistema [de referencia de coordenadas WGS-84](https://earth-info.nga.mil/GandG/update/index.php?action=home) .
> * El [geodésico de referencia](https://en.wikipedia.org/wiki/Geodetic_datum) que se utiliza para medir la distancia en la tierra es una esfera. Los bordes de línea son los poliedros en la esfera.

**Definición y restricciones de LineString**

Dynamic ({"type": "LineString", "Coordinates": [[lng_1, lat_1], [lng_2, lat_2],..., [lng_N, lat_N]]})

* La matriz de coordenadas LineString debe contener al menos dos entradas.
* Las coordenadas [longitud, latitud] deben ser válidas donde la longitud es un número real en el intervalo [-180, + 180] y la latitud es un número real en el intervalo [-90, + 90].
* La longitud del borde debe ser inferior a 180 grados. Se elegirá el borde más corto entre los dos vértices.

> [!TIP]
> Para mejorar el rendimiento, use líneas literales.

**Ejemplos**

En el ejemplo siguiente se busca la distancia más corta entre el aeropuerto de las Vegas y una carretera cercana.

:::image type="content" source="images/geo-distance-point-to-line-function/distance-point-to-line.png" alt-text="Distancia entre el aeropuerto de las Vegas y la carretera":::

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print distance_in_meters = geo_distance_point_to_line(-115.199625, 36.210419, dynamic({ "type":"LineString","coordinates":[[-115.115385,36.229195],[-115.136995,36.200366],[-115.140252,36.192470],[-115.143558,36.188523],[-115.144076,36.181954],[-115.154662,36.174483],[-115.166431,36.176388],[-115.183289,36.175007],[-115.192612,36.176736],[-115.202485,36.173439],[-115.225355,36.174365]]}))
```

| distance_in_meters |
|--------------------|
| 3797.88887253334   |

Eventos de Storm en la costa sur de EE. UU. Los eventos se filtran por una distancia máxima de 5 km desde la línea de tierra definida.

:::image type="content" source="images/geo-distance-point-to-line-function/us-south-coast-storm-events.png" alt-text="Eventos de Storm en la costa del sur de EE. UU.":::

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
let southCoast = dynamic({"type":"LineString","coordinates":[[-97.18505859374999,25.997549919572112],[-97.58056640625,26.96124577052697],[-97.119140625,27.955591004642553],[-94.04296874999999,29.726222319395504],[-92.98828125,29.82158272057499],[-89.18701171875,29.11377539511439],[-89.384765625,30.315987718557867],[-87.5830078125,30.221101852485987],[-86.484375,30.4297295750316],[-85.1220703125,29.6880527498568],[-84.00146484374999,30.14512718337613],[-82.6611328125,28.806173508854776],[-82.81494140625,28.033197847676377],[-82.177734375,26.52956523826758],[-80.9912109375,25.20494115356912]]});
StormEvents
| project BeginLon, BeginLat, EventType
| where geo_distance_point_to_line(BeginLon, BeginLat, southCoast) < 5000
| render scatterchart with (kind=map) // map rendering available in Kusto Explorer desktop
```

Recogida de taxis de NY. Las reactivaciones se filtran por la distancia máxima de 0,1 m de la línea definida.

:::image type="content" source="images/geo-distance-point-to-line-function/park-ave-ny-road.png" alt-text="Eventos de Storm en la costa del sur de EE. UU.":::

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
nyc_taxi
| project pickup_longitude, pickup_latitude
| where geo_distance_point_to_line(pickup_longitude, pickup_latitude, dynamic({"type":"LineString","coordinates":[[-73.97583961486816,40.75486445336327],[-73.95215034484863,40.78743027428638]]})) <= 0.1
| take 50
| render scatterchart with (kind=map) // map rendering available in Kusto Explorer desktop
```

En el ejemplo siguiente se devolverá un resultado null debido a la entrada LineString no válida.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print distance_in_meters = geo_distance_point_to_line(1,1, dynamic({ "type":"LineString"}))
```

| distance_in_meters |
|--------------------|
|                    |

En el ejemplo siguiente se devolverá un resultado null debido a la entrada de coordenadas no válida.

```kusto
print distance_in_meters = geo_distance_point_to_line(300, 3, dynamic({ "type":"LineString","coordinates":[[1,1],[2,2]]}))
```

| distance_in_meters |
|--------------------|
|                    |
