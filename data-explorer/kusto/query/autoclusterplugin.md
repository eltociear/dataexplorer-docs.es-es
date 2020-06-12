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
ms.openlocfilehash: bfa5286a03d06282682953a23c6b2a2705c58a9c
ms.sourcegitcommit: ae72164adc1dc8d91ef326e757376a96ee1b588d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2020
ms.locfileid: "84717077"
---
# <a name="autocluster-plugin"></a>complemento AutoCluster

```kusto
T | evaluate autocluster()
```

`autocluster`busca patrones comunes de atributos discretos (dimensiones) en los datos. A continuación, se reducen los resultados de la consulta original, ya sea 100 o 100 000 filas, a un número pequeño de patrones. El complemento se desarrolló para ayudar a analizar los errores (por ejemplo, excepciones o bloqueos), pero puede funcionar potencialmente en cualquier conjunto de datos filtrado.

**Sintaxis**

`T | evaluate autocluster(`*argumentos* de`)`

**Devuelve**

El `autocluster` complemento devuelve un conjunto de patrones (normalmente pequeño). Los patrones capturan partes de los datos con valores comunes compartidos entre varios atributos discretos. Cada patrón de los resultados se representa mediante una fila.

La primera columna es el identificador de segmento. Las dos columnas siguientes son el número y el porcentaje de filas fuera de la consulta original capturadas por el patrón. Las columnas restantes provienen de la consulta original. Su valor es un valor específico de la columna o un valor comodín (que, de forma predeterminada, es null) significa valores de variable.

Los patrones no son distintos, pueden estar superpuestos y normalmente no cubren todas las filas originales. Algunas filas no pueden estar en cualquier patrón.

> [!TIP]
> Use [Where](./whereoperator.md) y [Project](./projectoperator.md) en la canalización de entrada para reducir los datos a lo que le interesa.
>
> Al buscar una fila interesante, puede profundizar aún más mediante la adición de sus valores específicos a su filtro `where` .

**Argumentos (todos opcionales)**

`T | evaluate autocluster(`[*SizeWeight*, *WeightColumn*, *NumSeeds*, *CustomWildcard*, *CustomWildcard*,...]`)`

Todos los argumentos son opcionales, pero se deben ordenar como se indica anteriormente. Para indicar que se debe usar el valor predeterminado, coloque el valor de tilde de cadena ' ~ ' (vea la columna de ejemplo en la tabla).

|Argumento        | Tipo, intervalo, predeterminado              |Descripción                | Ejemplo                                        |
|----------------|-----------------------------------|---------------------------|------------------------------------------------|
| SizeWeight     | 0 < *doble* < 1 [valor predeterminado: 0,5]   | Proporciona un control sobre el equilibrio entre los valores genéricos (de alta cobertura) e informativos (muchos compartidos). Si aumenta el valor, normalmente se reduce el número de patrones, y cada patrón tiende a cubrir una cobertura de porcentaje mayor. Si disminuye el valor, normalmente se generan patrones más específicos con más valores compartidos y una cobertura de porcentaje más pequeña. La fórmula en el capó es una media geométrica ponderada entre la puntuación genérica normalizada y la puntuación informativa con pesos `SizeWeight` y`1-SizeWeight`                   | `T | evaluate autocluster(0.8)`                |
|WeightColumn    | *column_name*                     | Considera cada fila de la entrada según el peso especificado (de forma predeterminada cada fila tiene un peso de '1'). El argumento debe ser un nombre de una columna numérica (como int, Long, real). Un uso común de una columna de peso es tener en cuenta el muestreo o la creación de depósitos y la agregación de los datos que ya se han insertado en cada fila.                                                                                                       | `T | evaluate autocluster('~', sample_Count)` | 
| NumSeeds        | *int* [valor predeterminado: 25]              | El número de inicializaciones determina el número de puntos de búsqueda local iniciales del algoritmo. En algunos casos, en función de la estructura de los datos y si aumenta el número de semillas, el número (o la calidad) de los resultados aumenta a través del espacio de búsqueda expandido con un equilibrio de consulta más lento. El valor tiene resultados reducidos en ambas direcciones, por lo que si se reduce a menos de cinco, se lograrán mejoras de rendimiento insignificantes. Si aumenta a la versión anterior a la 50, rara vez generará patrones adicionales.                                         | `T | evaluate autocluster('~', '~', 15)`       |
| CustomWildcard  | *"any_value_per_type"*           | Establece el valor de carácter comodín para un tipo específico en la tabla de resultados. Indicará que el patrón actual no tiene ninguna restricción en esta columna. El valor predeterminado es null, ya que el valor predeterminado de la cadena es una cadena vacía. Si el valor predeterminado es un buen valor en los datos, se debe usar un valor comodín diferente (como `*` ).                                                                                                                | `T | evaluate autocluster('~', '~', '~', '*', int(-1), double(-1), long(0), datetime(1900-1-1))` |

## <a name="examples"></a>Ejemplos

### <a name="example"></a>Ejemplo

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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

### <a name="example-with-custom-wildcards"></a>Ejemplo con caracteres comodín personalizados

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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

## <a name="see-also"></a>Vea también

* [basket](./basketplugin.md)
* [reducen](./reduceoperator.md)

* `autocluster`se basa en gran medida en el algoritmo de expansión de inicialización del siguiente documento: [algoritmos para la minería de datos de telemetría con atributos discretos](https://www.scitepress.org/DigitalLibrary/PublicationsDetail.aspx?ID=d5kcrO+cpEU=&t=1). 
