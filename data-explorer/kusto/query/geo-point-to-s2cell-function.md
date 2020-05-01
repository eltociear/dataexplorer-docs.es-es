---
title: geo_point_to_s2cell ()-Explorador de datos de Azure | Microsoft Docs
description: En este artículo se describe geo_point_to_s2cell () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: d1cd2106865419f9407b3ff464d9852eb5ffb630
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82618483"
---
# <a name="geo_point_to_s2cell"></a>geo_point_to_s2cell()

Calcula el valor de cadena del token de celda S2 para una ubicación geográfica.

Para obtener más información sobre las celdas S2, haga clic [aquí](http://s2geometry.io/devguide/s2cell_hierarchy).

**Sintaxis**

`geo_point_to_s2cell(`*nivel de**latitud de*longitud*level* `, ``, ``)`

**Argumentos**

* *longitud*: valor de longitud de una ubicación geográfica. La longitud x se considerará válida si x es un número real y x está en el intervalo [-180, + 180]. 
* *latitud*: valor de latitud de una ubicación geográfica. La latitud y se considerará válida si y es un número real e y en el intervalo [-90, + 90]. 
* *LEVEL*: un opcional `int` que define el nivel de celda solicitado. Los valores admitidos están en el intervalo [0, 30]. Si no se especifica, se usa `11` el valor predeterminado.

**Devuelve**

Valor de cadena del token de celda S2 de una ubicación geográfica determinada. Si la coordenada o el nivel no son válidos, la consulta generará un resultado vacío.

> [!NOTE]
>
> * S2Cell puede ser una herramienta útil de agrupación en clústeres geoespaciales.
> * S2Cell tiene 31 niveles de jerarquía con cobertura de área que abarca desde 85011, 012.19 km ² en el nivel 0 hasta 00.44 cm ² en el nivel 30 más bajo.
> * S2Cell conserva el área del centro de la celda durante el aumento del nivel de 0 a 30.
> * S2Cell es una celda en una superficie de esfera y sus bordes son poliedros.
> * La invocación de la función [geo_s2cell_to_central_point ()](geo-s2cell-to-central-point-function.md) en una cadena de token de s2cell que se calculó en la longitud x y latitud y no devolverá necesariamente x e y.
> * Es posible que dos ubicaciones geográficas estén muy cerca unas de otras, pero que tengan distintos tokens de celda S2.

**Nivel de cobertura de área aproximada de celda S2 por valor de nivel**

Para cada nivel, el tamaño de s2cell es similar, pero no exactamente igual. El tamaño de las celdas cercanas tiende a ser más similar.

|Nivel|Longitud de borde de celda aleatoria mínima (RU)|Longitud máxima de borde de celda aleatorio (EE. UU.)|
|--|--|--|
|0|7842 km|7842 km|
|1|3921 km|5004 km|
|2|1825 km|2489 km|
|3|840 km|1310 km|
|4|432 km|636 km|
|5|210 km|315 km|
|6|108 km|156 km|
|7|54 km|78 km|
|8|27 km|39 km|
|9|14 km|20 km|
|10|7 km|10 km|
|11|3 km|5 km|
|12|1699 m|2 km|
|13|850 m|1225 m|
|14|425 m|613 m|
|15|212 m|306 m|
|16|106 m|153 m|
|17|53 m|77 m|
|18|27 m|38 m|
|19|13 m|19 m|
|20|7 m|10 m|
|21|3 m|5 m|
|22|166 cm|2 m|
|23|83 cm|120 cm|
|24|41 cm|60 cm|
|25|21 cm|30 cm|
|26|10 cm|15 cm|
|27|5 cm|7 cm|
|28|2 cm|4 cm|
|29|12 mm|18 mm|
|30|6 mm|9 mm|

[Aquí](http://s2geometry.io/resources/s2cell_statistics)puede encontrar el origen de la tabla.

Vea también [geo_point_to_geohash ()](geo-point-to-geohash-function.md).

**Ejemplos**

Eventos de Storm de EE. UU. agregados por s2cell.

:::image type="content" source="images/geo-point-to-s2cell-function/s2cell.png" alt-text="S2cell de EE. UU.":::

```kusto
StormEvents
| project BeginLon, BeginLat
| summarize by hash=geo_point_to_s2cell(BeginLon, BeginLat, 5)
| project geo_s2cell_to_central_point(hash)
| render scatterchart with (kind=map) // map rendering available in Kusto Explorer desktop
```

```kusto
print s2cell = geo_point_to_s2cell(-80.195829, 25.802215, 8)
```

| s2cell |
|--------|
| 88d9b  |

En el ejemplo siguiente se buscan grupos de coordenadas. Cada par de coordenadas del grupo reside en s2cell con el área máxima de 1632,45 km².
```kusto
datatable(location_id:string, longitude:real, latitude:real)
[
  "A", 10.1234, 53,
  "B", 10.3579, 53,
  "C", 10.6842, 53,
]
| summarize count = count(),                                        // items per group count
            locations = make_list(location_id)                      // items in the group
            by s2cell = geo_point_to_s2cell(longitude, latitude, 8) // s2 cell of the group
```

| s2cell | count | locations |
|--------|-------|-----------|
| 47b1d  | 2     | ["A", "B"] |
| 47ae3  | 1     | ["C"]     |

En el ejemplo siguiente se genera un resultado vacío debido a la entrada de coordenadas no válida.
```kusto
print s2cell = geo_point_to_s2cell(300,1,8)
```

| s2cell |
|--------|
|        |

En el ejemplo siguiente se genera un resultado vacío debido a la entrada de nivel no válida.
```kusto
print s2cell = geo_point_to_s2cell(1,1,35)
```

| s2cell |
|--------|
|        |

En el ejemplo siguiente se genera un resultado vacío debido a la entrada de nivel no válida.
```kusto
print s2cell = geo_point_to_s2cell(1,1,int(null))
```

| s2cell |
|--------|
|        |