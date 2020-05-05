---
title: 'complemento de diffpatterns: Azure Explorador de datos'
description: En este artículo se describe el complemento de diffpatterns en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 9931b297f5a86c46a8502902a6c396fbb2fd4191
ms.sourcegitcommit: 4f68d6dbfa6463dbb284de0aa17fc193d529ce3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82741730"
---
# <a name="diff-patterns-plugin"></a>complemento de patrones de comparación

Compara dos conjuntos de datos de la misma estructura y busca patrones de atributos discretos (dimensiones) que caracterizan las diferencias entre los dos conjuntos de datos.
 `Diffpatterns`se desarrolló para ayudar a analizar los errores (por ejemplo, mediante la comparación de errores con errores en un período de tiempo determinado), pero puede encontrar diferencias entre dos conjuntos de datos cualesquiera de la misma estructura. 

```kusto
T | evaluate diffpatterns(splitColumn)
```


**Sintaxis**

`T | evaluate diffpatterns(SplitColumn, SplitValueA, SplitValueB [, WeightColumn, Threshold, MaxDimensions, CustomWildcard, ...])` 

**Argumentos necesarios**

* SplitColumn: *nombre_columna*

    Indica al algoritmo cómo dividir la consulta en conjuntos de datos. Según los valores especificados para los argumentos SplitValueA y SplitValueB (ver a continuación), el algoritmo divide la consulta en dos conjuntos de datos, "A" y "B", y analiza las diferencias entre ellos. Por tanto, la columna dividida debe tener al menos dos valores distintos.

* SplitValueA - *cadena*

    Una representación de cadena de uno de los valores de SplitColumn que se especificó. Todas las filas que tienen este valor en su SplitColumn se consideran como conjunto de datos "A".

* SplitValueB - *cadena*

    Una representación de cadena de uno de los valores de SplitColumn que se especificó. Todas las filas que tienen este valor en su SplitColumn se consideran como conjunto de datos "B".

    Ejemplo: `T | extend splitColumn=iff(request_responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure") `

**Argumentos opcionales**

Todos los demás argumentos son opcionales, pero se deben ordenar como se indica a continuación. Para indicar que se debe usar el valor predeterminado, use el carácter de tilde: ~ (vea los ejemplos siguientes).

* WeightColumn - *nombre_columna*

    Considera cada fila de la entrada según el peso especificado (de forma predeterminada cada fila tiene un peso de '1'). El argumento debe ser un nombre de una columna numérica (por ejemplo, `int`, `long`, `real`).
    Un uso común de una columna de peso es tener en cuenta el muestreo o la creación de depósitos y la agregación de los datos que ya se han insertado en cada fila.
    
    Ejemplo: `T | extend splitColumn=iff(request_responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure", sample_Count) `

* Umbral-0,015 < *doble* < 1 [valor predeterminado: 0,05]

    Establece la diferencia mínima de los patrones (relación) entre los dos conjuntos.

    Ejemplo: `T | extend splitColumn = iff(request-responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure", "~", 0.04)`

* MaxDimensions-0 < *int* [valor predeterminado: Unlimited]

    Establece el número máximo de dimensiones no correlacionadas por patrón de resultado. Al especificar un límite, se reduce el tiempo de ejecución de la consulta.

    Ejemplo: `T | extend splitColumn = iff(request-responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure", "~", "~", 3)`

* CustomWildcard- *"any-Value-per-Type"*

    Establece el valor de carácter comodín para un tipo específico en la tabla de resultados que indicará que el patrón actual no tiene una restricción en esta columna.
    El valor predeterminado es null, y para las cadenas es una cadena vacía. Si el valor predeterminado es un valor viable en los datos, se debe usar un valor comodín distinto (por ejemplo `*`,).
    Vea el ejemplo siguiente.

    Ejemplo: `T | extend splitColumn = iff(request-responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure", "~", "~", "~", int(-1), double(-1), long(0), datetime(1900-1-1))`

**Devuelve**

`Diffpatterns`Devuelve un pequeño conjunto de patrones que capturan diferentes partes de los datos de los dos conjuntos (es decir, un patrón que captura un gran porcentaje de las filas del primer conjunto de datos y un porcentaje bajo de las filas del segundo conjunto). Cada patrón se representa mediante una fila en los resultados.

El resultado de `diffpatterns` devuelve las columnas siguientes:

* SegmentId: la identidad asignada al patrón en la consulta actual (Nota: no se garantiza que los identificadores sean los mismos en las consultas repetidas).

* CountA: el número de filas capturado por el patrón en el conjunto A (el conjunto A es el `where tostring(splitColumn) == SplitValueA`equivalente de).

* CountB: el número de filas capturado por el patrón en el conjunto B (el conjunto B es el `where tostring(splitColumn) == SplitValueB`equivalente de).

* Porcentaje: el porcentaje de filas de set A capturado por el patrón (100,0 * CountA/Count (establecer)).

* PercentB: el porcentaje de filas del conjunto B capturado por el patrón (100,0 * CountB/Count (SetB)).

* PercentDiffAB: diferencia de punto porcentual absoluta entre A y B (| Percenta-PercentB |) es la medida principal de la importancia de los patrones en la descripción de la diferencia entre los dos conjuntos.

* Resto de las columnas: son el esquema original de la entrada y describen el patrón, cada fila (patrón) vuelve a reenviar la intersección de los valores no comodín de las columnas (equivalente `where col1==val1 and col2==val2 and ... colN=valN` a para cada valor no comodín de la fila).

Para cada patrón, las columnas que no están establecidas en el patrón (es decir, sin restricciones en un valor específico) contendrán un valor comodín, que es null de forma predeterminada. Vea en la sección argumentos a continuación cómo se pueden cambiar manualmente los caracteres comodín.

* Nota: los patrones no suelen ser distintos. Pueden estar superpuestos y normalmente no cubren todas las filas originales. Algunas filas no pueden estar en cualquier patrón.


**Sugerencias**

Use [Where](./whereoperator.md) y [Project](./projectoperator.md) en la canalización de entrada para reducir los datos a lo que le interesa.

Al buscar una fila interesante, puede profundizar aún más mediante la adición de sus valores específicos a su filtro `where` .

* Nota: `diffpatterns` pretende encontrar patrones significativos (que capturen partes de la diferencia de datos entre los conjuntos) y no se destinan a diferencias fila por fila.

**Ejemplo**

```kusto
StormEvents 
| where monthofyear(StartTime) == 5
| extend Damage = iff(DamageCrops + DamageProperty > 0 , 1 , 0)
| project State , EventType , Source , Damage, DamageCrops
| evaluate diffpatterns(Damage, "0", "1" )
```

|SegmentId|CountA|CountB|PercentA|PercentB|PercentDiffAB|Estado|EventType|Source|DamageCrops|
|---|---|---|---|---|---|---|---|---|---|
|0|2278|93|49,8|7.1|42,7||Granizo||0|
|1|779|512|17,03|39,08|22,05||Viento de tormenta|||
|2|1098|118|24,01|9,01|15|||Observador entrenado|0|
|3|136|158|2,97|12.06|9,09|||Periódico||
|4|359|214|7,85|16,34|8,49||Riada|||
|5|50|122|1,09|9,31|8,22|IOWA||||
|6|655|279|14,32|21,3|6,98|||Cuerpos de seguridad||
|7|150|117|3,28|8,93|5,65||Inundación|||
|8|362|176|7,91|13,44|5,52|||Administrador de emergencia||
