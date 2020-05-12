---
title: 'complemento AutoCluster: Azure Explorador de datos'
description: En este artículo se describe el complemento AutoCluster de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0b2d76c0dd7a3ee0724db67aa8cb0cd9229748fa
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83225537"
---
# <a name="autocluster-plugin"></a>complemento AutoCluster

```kusto
T | evaluate autocluster()
```

AutoCluster busca patrones comunes de atributos discretos (dimensiones) en los datos y reduce los resultados de la consulta original (ya sean 100 o 100 000 filas) a un pequeño número de patrones. AutoCluster se ha desarrollado para ayudar a analizar los errores (por ejemplo, las excepciones o los bloqueos), pero puede trabajar en cualquier conjunto de datos filtrado. 

**Sintaxis**

`T | evaluate autocluster(`*argumentos* de`)`

**Devuelve**

AutoCluster devuelve un conjunto de patrones (normalmente pequeño) que captura partes de los datos con valores comunes compartidos entre varios atributos discretos. Cada patrón se representa mediante una fila en los resultados. 

La primera columna es el identificador de segmento. Las dos columnas siguientes son el recuento y el porcentaje de filas de la consulta original capturadas por el patrón. Las columnas restantes proceden de la consulta original y su valor es un valor específico de la columna o un valor comodín (que son NULL de forma predeterminada) entendidos como valores de las variables. 

Tenga en cuenta que los patrones no son distintivos: pueden solaparse y, normalmente, no cubren todas las filas originales. Algunas filas no pueden estar en cualquier patrón.

> [!TIP]
> Use [Where](./whereoperator.md) y [Project](./projectoperator.md) en la canalización de entrada para reducir los datos a lo que le interesa.
>
> Al buscar una fila interesante, puede profundizar aún más mediante la adición de sus valores específicos a su filtro `where` .

**Argumentos (todos opcionales)**

`T | evaluate autocluster(`[*SizeWeight*, *WeightColumn*, *NumSeeds*, *CustomWildcard*, *CustomWildcard*,...]`)`

Todos los argumentos son opcionales, pero se deben ordenar como se indica anteriormente. Para indicar que se debe usar el valor predeterminado, use el carácter de tilde: ~ (vea los ejemplos siguientes).

Argumentos disponibles:

* SizeWeight-0 < *doble* < 1 [valor predeterminado: 0,5]

    Proporciona control sobre el equilibrio entre genérico (gran cobertura) e informativo (muchos valores compartidos). Aumentar el valor normalmente reduce el número de patrones, y cada patrón tiende a abarcar un porcentaje mayor. Reducir el valor normalmente genera patrones más específicos, con más valores compartidos y una cobertura de porcentaje más pequeña. La fórmula del capó es una media geométrica ponderada entre la puntuación genérica normalizada y la puntuación `SizeWeight` `1-SizeWeight` informativa con y como pesos. 

    Ejemplo: `T | evaluate autocluster(0.8)`

* WeightColumn - *nombre_columna*

    Considera cada fila de la entrada según el peso especificado (de forma predeterminada cada fila tiene un peso de '1'). El argumento debe ser un nombre de una columna numérica (por ejemplo, int, long, real). Un uso común de una columna de peso es tener en cuenta el muestreo o la creación de depósitos y la agregación de los datos que ya se han insertado en cada fila.
    
    Ejemplo: `T | evaluate autocluster('~', sample_Count)` 

* NumSeeds- *int* [valor predeterminado: 25] 

    El número de inicializaciones determina el número de puntos de búsqueda local iniciales del algoritmo. En algunos casos, según la estructura de los datos, un número de inicializaciones creciente aumenta el número o la calidad de los resultados mediante un espacio de búsqueda mayor con un equilibrio de consulta más lento. El valor tiene unos resultados más reducidos en ambas direcciones, por lo que reducirlo por debajo de 5 logrará mejoras de rendimiento insignificantes y el aumento por encima de 50 rara vez generará patrones adicionales.

    Ejemplo:  `T | evaluate autocluster('~', '~', 15)`

* CustomWildcard- *"any_value_per_type"*

    Establece el valor de carácter comodín para un tipo específico en la tabla de resultados que indicará que el patrón actual no tiene una restricción en esta columna.
    El valor predeterminado es null, y para las cadenas es una cadena vacía. Si el valor predeterminado es un valor viable en los datos, se debe usar un valor comodín distinto (por ejemplo, `*` ).

    Ejemplo: `T | evaluate autocluster('~', '~', '~', '*', int(-1), double(-1), long(0), datetime(1900-1-1))`

**Ejemplo**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where monthofyear(StartTime) == 5
| extend Damage = iff(DamageCrops + DamageProperty > 0 , "YES" , "NO")
| project State , EventType , Damage
| evaluate autocluster(0.6)
```

|SegmentId|Count|Percent|Estado|EventType|Daños|
|---|---|---|---|---|---|---|---|---|
|0|2278|38,7||Granizo|No
|1|512|8,7||Viento de tormenta|SÍ
|2|898|15,3|TEXAS||

**Ejemplo con caracteres comodín personalizados**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where monthofyear(StartTime) == 5
| extend Damage = iff(DamageCrops + DamageProperty > 0 , "YES" , "NO")
| project State , EventType , Damage 
| evaluate autocluster(0.2, '~', '~', '*')
```

|SegmentId|Count|Percent|Estado|EventType|Daños|
|---|---|---|---|---|---|---|---|---|
|0|2278|38,7|\*|Granizo|No
|1|512|8,7|\*|Viento de tormenta|SÍ
|2|898|15,3|TEXAS|\*|\*

**Vea también**

* [basket](./basketplugin.md)
* [reducen](./reduceoperator.md)

**Información adicional**

* AutoCluster se basa en gran medida en el algoritmo de expansión de valores de inicialización del documento siguiente: [Algorithms for Telemetry Data Mining using Discrete Attributes](https://www.scitepress.org/DigitalLibrary/PublicationsDetail.aspx?ID=d5kcrO+cpEU=&t=1) (Algoritmos para minería de datos de telemetría con atributos discretos). 

