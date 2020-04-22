---
title: geo_point_in_circle() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe geo_point_in_circle() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/03/2020
ms.openlocfilehash: ca3ca8a1ac2299c43888ac827d46c25d02e4999d
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663799"
---
# <a name="geo_point_in_circle"></a>geo_point_in_circle()

Calcula si las coordenadas geoespaciales están dentro de un círculo en la Tierra.

**Sintaxis**

`geo_point_in_circle(`*p_longitude*`, `*p_latitude*`, `*c_radius* *pc_latitude*`, `*pc_latitude*`, `pc_longitude`)`

**Argumentos**

* *p_longitude*: Valor de longitud de coordenadas geoespaciales en grados. El valor válido es un número real y en el intervalo [-180, +180].
* *p_latitude*: Valor de latitud de coordenadas geoespaciales en grados. El valor válido es un número real y en el rango [-90, +90].
* *pc_longitude*: Valor de longitud de coordenadas geoespaciales del centro del círculo en grados. El valor válido es un número real y en el intervalo [-180, +180].
* *pc_latitude:* círculo centro de coordenadas geoespaciales valor de latitud en grados. El valor válido es un número real y en el rango [-90, +90].
* *c_radius*: Radio circular en metros. El valor válido debe ser positivo.

**Devuelve**

Indica si las coordenadas geoespaciales están dentro de un círculo. Si las coordenadas o el círculo no son válidos, la consulta producirá un resultado nulo.

> [!NOTE]
>* Las coordenadas geoespaciales se interpretan como representadas por el sistema de referencia de coordenadas [WGS-84.](https://earth-info.nga.mil/GandG/update/index.php?action=home)
>* El [dato geodésico](https://en.wikipedia.org/wiki/Geodetic_datum) utilizado para medir la distancia en la Tierra es una esfera.
>* El círculo es una tapa esférica en la Tierra. El radio de la tapa se mide a lo largo de la superficie de la esfera.

**Ejemplos**

La siguiente consulta busca todos los lugares en el área definida por un círculo con un radio de 18 km cuyo centro se encuentra en las coordenadas [-122.317404, 47.609119].

:::image type="content" source="images/queries/geo/circle_seattle.png" alt-text="Lugares cerca de Seattle":::

```kusto
datatable(longitude:real, latitude:real, place:string)
[
    real(-122.317404), 47.609119, 'Seattle',                   // In circle 
    real(-123.497688), 47.458098, 'Olympic National Forest',   // In exterior of circle  
    real(-122.201741), 47.677084, 'Kirkland',                  // In circle
    real(-122.443663), 47.247092, 'Tacoma',                    // In exterior of circle
    real(-122.121975), 47.671345, 'Redmond',                   // In circle
]
| where geo_point_in_circle(longitude, latitude, -122.317404, 47.609119, 18000)
| project place
```

|place|
|---|
|Seattle|
|Kirkland|
|Redmond|

Eventos de tormenta en Orlando. Los eventos se filtran por coordenadas de Orlando, dentro de 100 km y se agregan por tipo de evento y hash.

:::image type="content" source="images/queries/geo/orlando_storm_events.png" alt-text="Eventos de tormenta en Orlando":::

```kusto
StormEvents
| project BeginLon, BeginLat, EventType
| where geo_point_in_circle(BeginLon, BeginLat, real(-81.3891), 28.5346, 1000 * 100)
| summarize count() by EventType, hash = geo_point_to_s2cell(BeginLon, BeginLat)
| project geo_s2cell_to_central_point(hash), EventType, count_
| render piechart with (kind=map) // map rendering available in Kusto Explorer desktop
```

En el ejemplo siguiente se muestra NY Taxi pickups cerca de alguna ubicación y dentro de 10 metros. Las recogidas relevantes se agregan por hash.

:::image type="content" source="images/queries/geo/circle_junction.png" alt-text="NY Taxi cerca Recogidas":::

```kusto
nyc_taxi
| project pickup_longitude, pickup_latitude
| where geo_point_in_circle( pickup_longitude, pickup_latitude, real(-73.9928), 40.7429, 10)
| summarize by hash = geo_point_to_s2cell(pickup_longitude, pickup_latitude, 22)
| project geo_s2cell_to_central_point(hash)
| render scatterchart with (kind = map) // map rendering available in Kusto Explorer desktop
```

En el ejemplo siguiente se devolverá true.
```kusto
print in_circle = geo_point_in_circle(-122.143564, 47.535677, -122.100896, 47.527351, 3500)
```

|in_circle|
|---|
|1|

En el ejemplo siguiente se devolverá false.
```kusto
print in_circle = geo_point_in_circle(-122.137575, 47.630683, -122.100896, 47.527351, 3500)
```

|in_circle|
|---|
|0|

En el ejemplo siguiente se devolverá un resultado nulo debido a la entrada de coordenadas no válida.
```kusto
print in_circle = geo_point_in_circle(200, 1, 1, 1, 1)
```

|in_circle|
|---|
||

En el ejemplo siguiente se devolverá un resultado nulo debido a la entrada de radio de círculo no válido.
```kusto
print in_circle = geo_point_in_circle(1, 1, 1, 1, -1)
```

|in_circle|
|---|
||