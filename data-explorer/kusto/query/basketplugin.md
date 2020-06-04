---
title: 'complemento de cesta: Azure Explorador de datos'
description: En este artículo se describe el complemento de cesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 05/26/2019
ms.openlocfilehash: a43275aa6d2938631cad052cfbdd9a185db487b2
ms.sourcegitcommit: 8953d09101f4358355df60ab09e55e71bc255ead
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2020
ms.locfileid: "84420856"
---
# <a name="basket-plugin"></a>complemento Basket

```kusto
T | evaluate basket()
```

basket encuentra todos los patrones frecuentes de atributos discretos (dimensiones) en los datos y devuelve todos los patrones frecuentes que pasan el umbral de frecuencia de la consulta original. Se garantiza que la cesta encuentra todos los patrones frecuentes de los datos, pero no se garantiza que tengan un tiempo de ejecución Polinómico, el tiempo de ejecución de la consulta es lineal en el número de filas pero, en algunos casos, puede ser exponencial en el número de columnas (dimensiones). basket se basa en el algoritmo Apriori desarrollado originalmente la para minería de datos de análisis de la cesta de la compra.

**Sintaxis**

`T | evaluate basket(`*argumentos* de`)`

**Devuelve**

Basket devuelve todos los patrones frecuentes que aparecen por encima del umbral de relación (valor predeterminado: 0,05) de las filas. Cada patrón se representa mediante una fila en los resultados.

La primera columna es el identificador de segmento. Las dos columnas siguientes son el recuento y el porcentaje de filas de la consulta original capturadas por el patrón. Las columnas restantes proceden de la consulta original y su valor es un valor específico de la columna o un valor comodín (que son NULL de forma predeterminada) entendidos como valores de las variables.

**Argumentos (todos opcionales)**

`T | evaluate basket(`[*Threshold*, *WeightColumn*, *MaxDimensions*, *CustomWildcard*, *CustomWildcard*,...]`)`

Todos los argumentos son opcionales, pero se deben ordenar como se indica anteriormente. Para indicar que se debe usar el valor predeterminado, use el carácter de tilde: ~ (vea los ejemplos siguientes).

Argumentos disponibles:

* Umbral-0,015 < *doble* < 1 [valor predeterminado: 0,05]

    Establece la relación mínima de las filas para que se consideren frecuentes (no se devolverán los patrones con una relación menor).
    
    Ejemplo: `T | evaluate basket(0.02)`

* WeightColumn - *nombre_columna*

    Considera cada fila de la entrada según el peso especificado (de forma predeterminada cada fila tiene un peso de '1'). El argumento debe ser un nombre de una columna numérica (por ejemplo, int, long, real). Un uso común de una columna de peso es tener en cuenta el muestreo o la creación de depósitos y la agregación de los datos que ya se han insertado en cada fila.
    
    Ejemplo: `T | evaluate basket('~', sample_Count)`

* MaxDimensions-1 < *int* [valor predeterminado: 5]

    Establece el número máximo de dimensiones no correlacionadas por basket, limitado de forma predeterminada para reducir el tiempo de ejecución de la consulta.

    Ejemplo: `T | evaluate basket('~', '~', 3)`

* CustomWildcard- *"any_value_per_type"*

    Establece el valor de carácter comodín para un tipo específico en la tabla de resultados que indicará que el patrón actual no tiene una restricción en esta columna.
    El valor predeterminado es null, y para las cadenas es una cadena vacía. Si el valor predeterminado es un valor viable en los datos, se debe usar un valor comodín distinto (por ejemplo, `*` ).
    Vea el ejemplo siguiente.

    Ejemplo: `T | evaluate basket('~', '~', '~', '*', int(-1), double(-1), long(0), datetime(1900-1-1))`

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where monthofyear(StartTime) == 5
| extend Damage = iff(DamageCrops + DamageProperty > 0 , "YES" , "NO")
| project State, EventType, Damage, DamageCrops
| evaluate basket(0.2)
```

|SegmentId|Count|Percent|State|EventType|Daños|DamageCrops|
|---|---|---|---|---|---|---|---|---|
|0|4574|77,7|||No|0
|1|2278|38,7||Granizo|No|0
|2|5675|96,4||||0
|3|2371|40,3||Granizo||0
|4|1279|21,7||Viento de tormenta||0
|5|2468|41,9||Granizo||
|6|1310|22,3|||SÍ|
|7|1291|21,9||Viento de tormenta||

**Ejemplo con caracteres comodín personalizados**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where monthofyear(StartTime) == 5
| extend Damage = iff(DamageCrops + DamageProperty > 0 , "YES" , "NO")
| project State, EventType, Damage, DamageCrops
| evaluate basket(0.2, '~', '~', '*', int(-1))
```

|SegmentId|Count|Percent|State|EventType|Daños|DamageCrops|
|---|---|---|---|---|---|---|---|---|
|0|4574|77,7|\*|\*|No|0
|1|2278|38,7|\*|Granizo|No|0
|2|5675|96,4|\*|\*|\*|0
|3|2371|40,3|\*|Granizo|\*|0
|4|1279|21,7|\*|Viento de tormenta|\*|0
|5|2468|41,9|\*|Granizo|\*|-1
|6|1310|22,3|\*|\*|SÍ|-1
|7|1291|21,9|\*|Viento de tormenta|\*|-1
