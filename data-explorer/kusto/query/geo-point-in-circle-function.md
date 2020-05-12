---
title: 'geo_point_in_circle (): Explorador de datos de Azure'
description: En este artículo se describe geo_point_in_circle () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: mbrichko
ms.service: data-explorer
ms.topic: reference
ms.date: 02/03/2020
ms.openlocfilehash: 6e6ef40fcdeb4942dc0924c86862ee8f6222ac12
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227194"
---
# <a name="geo_point_in_circle"></a>geo_point_in_circle()

Calcula si las coordenadas geoespaciales están dentro de un círculo en la tierra.

**Sintaxis**

`geo_point_in_circle(`*p_longitude* `, ` *p_latitude* `, ` *pc_longitude* `, ` *pc_latitude* `, ` *c_radius*`)`

**Argumentos**

* *p_longitude*: valor de longitud de la coordenada geoespacial en grados. El valor válido es un número real y en el intervalo [-180, + 180].
* *p_latitude*: valor de latitud de coordenadas geoespaciales en grados. El valor válido es un número real y en el intervalo [-90, + 90].
* *pc_longitude*: valor de longitud de la coordenada geoespacial del centro del círculo en grados. El valor válido es un número real y en el intervalo [-180, + 180].
* *pc_latitude*: valor de latitud de la coordenada geoespacial del centro del círculo en grados. El valor válido es un número real y en el intervalo [-90, + 90].
* *c_radius*: Radio circular en metros. El valor válido debe ser positivo.

**Devuelve**

Indica si las coordenadas geoespaciales están dentro de un círculo. Si las coordenadas o el círculo no son válidos, la consulta generará un resultado nulo.

> [!NOTE]
>* Las coordenadas geoespaciales se interpretan como representadas por el sistema [de referencia de coordenadas WGS-84](https://earth-info.nga.mil/GandG/update/index.php?action=home) .
>* El [geodésico de referencia](https://en.wikipedia.org/wiki/Geodetic_datum) que se utiliza para medir la distancia en la tierra es una esfera.
>* Un círculo es un extremo esférico en la tierra. El radio del extremo se mide a lo largo de la superficie de la esfera.

**Ejemplos**

La siguiente consulta busca todos los lugares en el área definida por el siguiente círculo: radio de 18 km, centro en las coordenadas [-122,317404, 47,609119].

:::image type="content" source="images/geo-point-in-circle-function/circle-seattle.png" alt-text="Lugares cerca de Seattle":::

<!-- csl: https://help.kusto.windows.net/Samples -->
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

Eventos de Storm en Orlando. Los eventos se filtran por 100 km dentro de las coordenadas del Orlando y se agregan por tipo de evento y hash.

:::image type="content" source="images/geo-point-in-circle-function/orlando-storm-events.png" alt-text="Eventos de Storm en Orlando":::

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
StormEvents
| project BeginLon, BeginLat, EventType
| where geo_point_in_circle(BeginLon, BeginLat, real(-81.3891), 28.5346, 1000 * 100)
| summarize count() by EventType, hash = geo_point_to_s2cell(BeginLon, BeginLat)
| project geo_s2cell_to_central_point(hash), EventType, count_
| render piechart with (kind=map) // map rendering available in Kusto Explorer desktop
```

En el ejemplo siguiente se muestra la recogida de taxi de NY en 10 metros de una ubicación determinada. Las reactivaciones relevantes se agregan por hash.

:::image type="content" source="images/geo-point-in-circle-function/circle-junction.png" alt-text="Recogidas cercanas de taxis de NY":::

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
nyc_taxi
| project pickup_longitude, pickup_latitude
| where geo_point_in_circle( pickup_longitude, pickup_latitude, real(-73.9928), 40.7429, 10)
| summarize by hash = geo_point_to_s2cell(pickup_longitude, pickup_latitude, 22)
| project geo_s2cell_to_central_point(hash)
| render scatterchart with (kind = map) // map rendering available in Kusto Explorer desktop
```

En el ejemplo siguiente se devolverá True.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print in_circle = geo_point_in_circle(-122.143564, 47.535677, -122.100896, 47.527351, 3500)
```

|in_circle|
|---|
|1|

En el ejemplo siguiente se devolverá FALSE.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print in_circle = geo_point_in_circle(-122.137575, 47.630683, -122.100896, 47.527351, 3500)
```

|in_circle|
|---|
|0|

En el ejemplo siguiente se devolverá un resultado null debido a la entrada de coordenadas no válida.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print in_circle = geo_point_in_circle(200, 1, 1, 1, 1)
```

|in_circle|
|---|
||

En el ejemplo siguiente se devolverá un resultado null debido a la entrada de radio del círculo no válido.

```kusto
print in_circle = geo_point_in_circle(1, 1, 1, 1, -1)
```

|in_circle|
|---|
||
