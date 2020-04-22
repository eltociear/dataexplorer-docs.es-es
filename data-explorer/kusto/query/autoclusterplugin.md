---
title: complemento de autocluster - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe el complemento de clúster automático en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: bbc89f8214b5d64cdc605d1771da7c27064cc629
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663904"
---
# <a name="autocluster-plugin"></a>plugin autocluster

```kusto
T | evaluate autocluster()
```

AutoCluster busca patrones comunes de atributos discretos (dimensiones) en los datos y reduce los resultados de la consulta original (ya sean 100 o 100 000 filas) a un pequeño número de patrones. AutoCluster se ha desarrollado para ayudar a analizar los errores (por ejemplo, las excepciones o los bloqueos), pero puede trabajar en cualquier conjunto de datos filtrado. 

**Sintaxis**

`T | evaluate autocluster(`*argumentos*`)`

**Devuelve**

AutoCluster devuelve un conjunto de patrones (normalmente pequeño) que captura partes de los datos con valores comunes compartidos entre varios atributos discretos. Cada patrón se representa mediante una fila en los resultados. 

La primera columna es el identificador de segmento. Las dos columnas siguientes son el recuento y el porcentaje de filas de la consulta original capturadas por el patrón. Las columnas restantes proceden de la consulta original y su valor es un valor específico de la columna o un valor comodín (que son NULL de forma predeterminada) entendidos como valores de las variables. 

Tenga en cuenta que los patrones no son distintivos: pueden solaparse y, normalmente, no cubren todas las filas originales. Algunas filas no pueden estar en cualquier patrón.

> [!TIP]
> Utilice [where](./whereoperator.md) y [project](./projectoperator.md) en la tubería de entrada para reducir los datos a lo que le interesa.
>
> Al buscar una fila interesante, puede profundizar aún más mediante la adición de sus valores específicos a su filtro `where` .

**Argumentos (todos opcionales)**

`T | evaluate autocluster(`[*SizeWeight*, *WeightColumn*, *NumSeeds*, *CustomWildcard*, *CustomWildcard*, ...]`)`

Todos los argumentos son opcionales, pero se deben ordenar como se indica anteriormente. Para indicar que se debe usar el valor predeterminado, use el carácter de tilde: ~ (vea los ejemplos siguientes).

Argumentos disponibles:

* Peso de tamaño - 0 < *doble* < 1 [predeterminado: 0,5]

    Proporciona control sobre el equilibrio entre genérico (gran cobertura) e informativo (muchos valores compartidos). Aumentar el valor normalmente reduce el número de patrones, y cada patrón tiende a abarcar un porcentaje mayor. Reducir el valor normalmente genera patrones más específicos, con más valores compartidos y una cobertura de porcentaje más pequeña. La fórmula bajo el capó es una media geométrica ponderada `SizeWeight` `1-SizeWeight` entre la puntuación genérica normalizada y la puntuación informativa con y como los pesos. 

    Ejemplo: `T | evaluate autocluster(0.8)`

* WeightColumn - *nombre_columna*

    Considera cada fila de la entrada según el peso especificado (de forma predeterminada cada fila tiene un peso de '1'). El argumento debe ser un nombre de una columna numérica (por ejemplo, int, long, real). Un uso común de una columna de peso es tener en cuenta el muestreo o la creación de depósitos y la agregación de los datos que ya se han insertado en cada fila.
    
    Ejemplo: `T | evaluate autocluster('~', sample_Count)` 

* NumSeeds - *int* [predeterminado: 25] 

    El número de inicializaciones determina el número de puntos de búsqueda local iniciales del algoritmo. En algunos casos, según la estructura de los datos, un número de inicializaciones creciente aumenta el número o la calidad de los resultados mediante un espacio de búsqueda mayor con un equilibrio de consulta más lento. El valor tiene resultados decrecientes en ambas direcciones por lo que disminuirlo por debajo de 5 logrará mejoras de rendimiento insignificantes y el aumento por encima de 50 rara vez generará patrones adicionales.

    Ejemplo: `T | evaluate autocluster('~', '~', 15)`

* CustomWildcard - *"any_value_per_type"*

    Establece el valor de carácter comodín para un tipo específico en la tabla de resultados que indicará que el patrón actual no tiene una restricción en esta columna.
    El valor predeterminado es null, y para las cadenas es una cadena vacía. Si el valor predeterminado es un valor viable en los datos, `*`se debe utilizar un valor comodín diferente (por ejemplo, ).

    Ejemplo: `T | evaluate autocluster('~', '~', '~', '*', int(-1), double(-1), long(0), datetime(1900-1-1))`

**Ejemplo**

```kusto
StormEvents 
| where monthofyear(StartTime) == 5
| extend Damage = iff(DamageCrops + DamageProperty > 0 , "YES" , "NO")
| project State , EventType , Damage
| evaluate autocluster(0.6)
```

|SegmentId|Count|Percent|State|EventType|Daños|
|---|---|---|---|---|---|---|---|---|
|0|2278|38,7||Granizo|No
|1|512|8,7||Viento de tormenta|SÍ
|2|898|15,3|TEXAS||

**Ejemplo con caracteres comodín personalizados**

```kusto
StormEvents 
| where monthofyear(StartTime) == 5
| extend Damage = iff(DamageCrops + DamageProperty > 0 , "YES" , "NO")
| project State , EventType , Damage 
| evaluate autocluster(0.2, '~', '~', '*')
```

|SegmentId|Count|Percent|State|EventType|Daños|
|---|---|---|---|---|---|---|---|---|
|0|2278|38,7|\*|Granizo|No
|1|512|8,7|\*|Viento de tormenta|SÍ
|2|898|15,3|TEXAS|\*|\*

**Vea también**

* [basket](./basketplugin.md)
* [Reducir](./reduceoperator.md)

**Información adicional**

* AutoCluster se basa en gran medida en el algoritmo de expansión de valores de inicialización del documento siguiente: [Algorithms for Telemetry Data Mining using Discrete Attributes](https://www.scitepress.org/DigitalLibrary/PublicationsDetail.aspx?ID=d5kcrO+cpEU=&t=1) (Algoritmos para minería de datos de telemetría con atributos discretos). 

