---
title: 'geo_point_to_geohash (): Explorador de datos de Azure'
description: En este artículo se describe geo_point_to_geohash () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: mbrichko
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: c37789ac490814288c7331f0b1ae86b8b2178d67
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226931"
---
# <a name="geo_point_to_geohash"></a>geo_point_to_geohash()

Calcula el valor de cadena geohash para una ubicación geográfica.

Obtenga más información sobre [geohash](https://en.wikipedia.org/wiki/Geohash).  

**Sintaxis**

`geo_point_to_geohash(`*longitud* `, ` *latitud* `, ` [*precisión*]`)`

**Argumentos**

* *longitud*: valor de longitud de una ubicación geográfica. La longitud x se considerará válida si x es un número real y está en el intervalo [-180, + 180]. 
* *latitud*: valor de latitud de una ubicación geográfica. La latitud y se considerará válida si y es un número real e y se encuentra en el intervalo [-90, + 90]. 
* *precisión*: un opcional `int` que define la precisión solicitada. Los valores admitidos se encuentran en el intervalo [1, 18]. Si no se especifica, se usa el valor predeterminado `5` .

**Devuelve**

Valor de cadena geohash de una ubicación geográfica determinada con la longitud de precisión solicitada. Si la coordenada o la precisión no son válidas, la consulta generará un resultado vacío.

> [!NOTE]
>
> * Geohash puede ser una herramienta útil de agrupación en clústeres geoespaciales.
> * Geohash tiene 18 niveles de precisión con cobertura de área comprendida entre 25 millones km² en el nivel 1 a 0,6 μ ² en el nivel 18 más bajo.
> * Los prefijos comunes de geohash indican proximidad de puntos entre sí. Cuanto más larga sea un prefijo compartido, más cerca de los dos lugares. El valor de precisión se traduce en la longitud de la geohash.
> * Geohash es un área rectangular en una superficie de plano.
> * La invocación de la función [geo_geohash_to_central_point ()](geo-geohash-to-central-point-function.md) en una cadena geohash que se calculó en la longitud x y latitud y no devolverá necesariamente x e y.
> * Debido a la definición de la geohash, es posible que dos ubicaciones geográficas estén muy cercanas entre sí, pero que tengan códigos geohash diferentes.

**Cobertura de área rectangular de geohash por valor de precisión:**

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
| 11       | 149,01 mm | 149,01 mm |
| 12       | 37,25 mm  | 18,63 mm  |
| 13       | 4,66 mm   | 4,66 mm   |
| 14       | 1,16 mm   | 0,58 mm   |
| 15       | 145,52 μ  | 145,52 μ  |
| 16       | 36,28 μ   | 18,19 μ   |
| 17       | 4,55 μ    | 4,55 μ    |
| 18       | 1,14 μ    | 0,57 μ    |

Vea también [geo_point_to_s2cell ()](geo-point-to-s2cell-function.md).

**Ejemplos**

Eventos de Storm de EE. UU. agregados por el algoritmo hash.

:::image type="content" source="images/geo-point-to-geohash-function/geohash.png" alt-text="Geohash de EE. UU.":::

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
StormEvents
| project BeginLon, BeginLat
| summarize by hash=geo_point_to_geohash(BeginLon, BeginLat, 3)
| project geo_geohash_to_central_point(hash)
| render scatterchart with (kind=map) // map rendering available in Kusto Explorer desktop
```

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print geohash = geo_point_to_geohash(139.806115, 35.554128, 12)  
```

| geohash      |
|--------------|
| xn76m27ty9g4 |

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print geohash = geo_point_to_geohash(-80.195829, 25.802215, 8)
```

|geohash|
|---|
|dhwfz15h|

En el ejemplo siguiente se buscan grupos de coordenadas. Cada par de coordenadas del grupo reside en un área rectangular de 4,88 km por 4,88 km.

<!-- csl: https://help.kusto.windows.net/Samples -->
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

En el ejemplo siguiente se genera un resultado vacío debido a la entrada de coordenadas no válida.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print geohash = geo_point_to_geohash(200,1,8)
```

| geohash |
|---------|
|         |

En el ejemplo siguiente se genera un resultado vacío debido a la entrada de precisión no válida.

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
print geohash = geo_point_to_geohash(1,1,int(null))
```

| geohash |
|---------|
|         |
