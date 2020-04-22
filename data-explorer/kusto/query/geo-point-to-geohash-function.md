---
title: geo_point_to_geohash() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe geo_point_to_geohash() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: b69d22c56cef4b54a99aa9aa3e9897a2ef177834
ms.sourcegitcommit: 29018b3db4ea7d015b1afa65d49ecf918cdff3d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "82030127"
---
# <a name="geo_point_to_geohash"></a>geo_point_to_geohash()

Calcula el valor de cadena Geohash para una ubicación geográfica.

Para obtener más información sobre Geohash, consulte [aquí](https://en.wikipedia.org/wiki/Geohash).  

**Sintaxis**

`geo_point_to_geohash(`*longitude*`, `*latitude*latitud`, `longitud [*precisión*]`)`

**Argumentos**

* *longitud*: Valor de longitud de una ubicación geográfica. Longitud x se considerará válida si x es un número real y está en el rango [-180, +180]. 
* *latitud*: Valor de latitud de una ubicación geográfica. La latitud y se considerará válida si y es un número real y y está en el rango [-90, +90]. 
* *precisión*: Un `int` opcional que define la precisión solicitada. Los valores admitidos están en el intervalo [1,18]. Si no se especifica, `5` se utiliza el valor predeterminado.

**Devuelve**

El valor de cadena Geohash de una ubicación geográfica determinada con la longitud de precisión solicitada. Si la coordenada o la precisión no son válidas, la consulta producirá un resultado vacío.

> [!NOTE]
>
> * Geohash puede ser una herramienta útil de agrupación en clústeres geoespaciales.
> * Geohash tiene 18 niveles de precisión con una cobertura de área que oscila entre los 25 millones de km2 en el nivel más alto 1 y 0,6 2 en el nivel más bajo 18.
> * El prefijo común de Geohashes indica en la proximidad de puntos entre sí. Cuanto más tiempo sea un prefijo compartido, más cerca estarán los dos lugares. El valor de precisión se traduce en longitud de geohash.
> * Geohash es un área rectangular en una superficie plana.
> * Invocar la función [geo_geohash_to_central_point()](geo-geohash-to-central-point-function.md) en una cadena de geohash que se calculó en longitud x y latitud y no necesariamente devolverá x e y.
> * Debido a la definición de Geohash, es posible que dos ubicaciones geográficas estén muy cerca una sin otras, pero tengan códigos Geohash diferentes.

**Cobertura de área rectangular Geohash por valor de precisión:**

| Precisión | Ancho     | Alto    |
|----------|-----------|-----------|
| 1        | 5000 km   | 5000 km   |
| 2        | 1250 km   | 625 km    |
| 3        | 156,25 km | 156,25 km |
| 4        | 39,06 km  | 19,53 km  |
| 5        | 4,88 km   | 4,88 km   |
| 6        | 1,22 km   | 0,61 km   |
| 7        | 152,59 m  | 152,59 m  |
| 8        | 38,15 m   | 19,07 m   |
| 9        | 4,77 m    | 4,77 m    |
| 10       | 1,19 m    | 0,59 m    |
| 11       | 149.01 mm | 149.01 mm |
| 12       | 37.25 mm  | 18.63 mm  |
| 13       | 4,66 mm   | 4,66 mm   |
| 14       | 1.16 mm   | 0,58 mm   |
| 15       | 145,52 ?  | 145,52 ?  |
| 16       | 36,28 o   | 18.19   |
| 17       | 4.55o    | 4.55o    |
| 18       | 1.14o    | 0,57 o    |

Véase también [geo_point_to_s2cell()](geo-point-to-s2cell-function.md).

**Ejemplos**

Eventos de tormenta de EE. UU. agregados por geohash.

:::image type="content" source="images/geo-point-to-geohash-function/geohash.png" alt-text="Geohash de EE. UU.":::

```kusto
StormEvents
| project BeginLon, BeginLat
| summarize by hash=geo_point_to_geohash(BeginLon, BeginLat, 3)
| project geo_geohash_to_central_point(hash)
| render scatterchart with (kind=map) // map rendering available in Kusto Explorer desktop
```

```kusto
print geohash = geo_point_to_geohash(139.806115, 35.554128, 12)  
```

| geohash      |
|--------------|
| xn76m27ty9g4 |

```kusto
print geohash = geo_point_to_geohash(-80.195829, 25.802215, 8)
```

|geohash|
|---|
|dhwfz15h|

En el ejemplo siguiente se encuentran grupos de coordenadas. Cada par de coordenadas del grupo residen en un área rectangular de 4,88 km en 4,88 km.
```kusto
datatable(location_id:string, longitude:real, latitude:real)
[
  "A", double(-122.303404), 47.570482,
  "B", double(-122.304745), 47.567052,
  "C", double(-122.278156), 47.566936,
]
| summarize count = count(),                                          // items per group count
            locations = make_list(location_id)                        // items in the group
            by geohash = geo_point_to_geohash(longitude, latitude)    // geohash of the group
```

| geohash | count | locations  |
|---------|-------|------------|
| c23n8   | 2     | ["A", "B"] |
| c23n9   | 1     | ["C"]      |

En el ejemplo siguiente se produce un resultado vacío debido a la entrada de coordenadas no válida.
```kusto
print geohash = geo_point_to_geohash(200,1,8)
```

| geohash |
|---------|
|         |

En el ejemplo siguiente se produce un resultado vacío debido a la entrada de precisión no válida.
```kusto
print geohash = geo_point_to_geohash(1,1,int(null))
```

| geohash |
|---------|
|         |