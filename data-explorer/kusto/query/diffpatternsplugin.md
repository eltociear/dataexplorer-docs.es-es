---
title: plugin diffpatterns - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe el complemento diffpatterns en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: fc4d1c7441e02eeeb1d90e1f8c0f2a521e3793da
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516004"
---
# <a name="diffpatterns-plugin"></a>diffpatterns plugin

```kusto
T | evaluate diffpatterns(splitColumn)
```
Compara dos conjuntos de datos de la misma estructura y busca patrones de atributos discretos (dimensiones) que caracterizan las diferencias entre los dos conjuntos de datos. diffpatterns se ha desarrollado para ayudar a analizar los errores (por ejemplo, al comparar errores y no errores durante un período determinado), pero puede encontrar las diferencias entre los dos conjuntos de datos de la misma estructura. 

**Sintaxis**

`T | evaluate diffpatterns(`SplitColumn, SplitValueA, SplitValueB [, WeightColumn, Threshold, MaxDimensions, CustomWildcard, ...]`)` 

**Argumentos requeridos**

* SplitColumn: *nombre_columna*

    Indica al algoritmo cómo dividir la consulta en conjuntos de datos. Según los valores especificados para los argumentos SplitValueA y SplitValueB (ver a continuación), el algoritmo divide la consulta en dos conjuntos de datos, "A" y "B", y analiza las diferencias entre ellos. Por tanto, la columna dividida debe tener al menos dos valores distintos.

* SplitValueA - *cadena*

    Una representación de cadena de uno de los valores de SplitColumn que se especificó. Todas las filas que tienen este valor en SplitColumn se consideran como el conjunto de datos "A".

* SplitValueB - *cadena*

    Una representación de cadena de uno de los valores de SplitColumn que se especificó. Todas las filas que tienen este valor en su SplitColumn considerado como conjunto de datos "B".

    Ejemplo: `T | extend splitColumn=iff(request_responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure") `

**Argumentos opcionales**

Todos los demás argumentos son opcionales, pero se deben ordenar como se indica a continuación. Para indicar que se debe usar el valor predeterminado, use el carácter de tilde: ~ (vea los ejemplos siguientes).

* WeightColumn - *nombre_columna*

    Considera cada fila de la entrada según el peso especificado (de forma predeterminada cada fila tiene un peso de '1'). El argumento debe ser un nombre de una columna numérica (por ejemplo, int, long, real).
    Un uso común de una columna de peso es tener en cuenta el muestreo o la creación de depósitos y la agregación de los datos que ya se han insertado en cada fila.
    
    Ejemplo: `T | extend splitColumn=iff(request_responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure", sample_Count) `

* Umbral - 0.015 < *doble* < 1 [predeterminado: 0.05]

    Establece la diferencia mínima de los patrones (relación) entre los dos conjuntos.

    Ejemplo: `T | extend splitColumn = iff(request-responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure", "~", 0.04)`

* MaxDimensions - 0 < *int* [predeterminado]

    Establece el número máximo de dimensiones no correlacionadas por patrón de resultados, especificando un límite que reduce el tiempo de ejecución de la consulta.

    Ejemplo: `T | extend splitColumn = iff(request-responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure", "~", "~", 3)`

* CustomWildcard - *"cualquier valor por tipo"*

    Establece el valor de carácter comodín para un tipo específico en la tabla de resultados que indicará que el patrón actual no tiene una restricción en esta columna.
    El valor predeterminado es null, y para las cadenas es una cadena vacía. Si el valor predeterminado es un valor viable en los datos, `*`se debe utilizar un valor comodín diferente (por ejemplo, ).
    Vea el ejemplo siguiente.

    Ejemplo: `T | extend splitColumn = iff(request-responseCode == 200, "Success" , "Failure") | evaluate diffpatterns(splitColumn, "Success","Failure", "~", "~", "~", int(-1), double(-1), long(0), datetime(1900-1-1))`

**Devuelve**

diffpatterns devuelve un conjunto (normalmente pequeño) de patrones que captura distintas partes de los datos en los dos conjuntos (es decir, un patrón que captura un gran porcentaje de las filas del primer conjunto de datos y un porcentaje bajo de las filas del segundo conjunto). Cada patrón se representa mediante una fila en los resultados.

El resultado de diffpatterns devuelve las siguientes columnas:

* SegmentId: el identificador asignado al patrón en la consulta actual (nota: no se garantiza que los identificadores sean los mismos en las consultas repetidas).

* CountA: el número de filas capturadas por el patrón `where tostring(splitColumn) == SplitValueA`en el conjunto A (el conjunto A es el equivalente de ).

* CountB: el número de filas capturadas por el patrón `where tostring(splitColumn) == SplitValueB`en el conjunto B (el conjunto B es el equivalente a ).

* PercentA: el porcentaje de filas en el conjunto A capturado por el patrón ( 100.0 * CountA / count(SetA) ).

* PercentB: el porcentaje de filas del conjunto B capturados por el patrón ( 100.0 * CountB / count(SetB) ).

* PercentDiffAB: la diferencia de punto porcentual absoluto entre A y B ( PercentA - PercentB ) es la principal medida de importancia de los patrones para describir la diferencia entre los dos conjuntos.

* Resto de las columnas: son el esquema original de la entrada y describen el patrón, cada fila (patrón) resiente la intersección de los valores no comodín de las columnas (equivalente `where col1==val1 and col2==val2 and ... colN=valN` a cada valor no comodín de la fila).

Para cada patrón, las columnas que no se establecen en el patrón (es decir, sin restricción en un valor específico) contendrán un valor comodín que es nulo de forma predeterminada (consulte en la sección Argumentos debajo de cómo se pueden cambiar manualmente los caracteres comodín).

* Nota: los patrones generalmente no son distintos, pueden estar superpuestos y por lo general no cubren todas las filas originales. Algunas filas no pueden estar en cualquier patrón.


**Sugerencias**

Utilice [where](./whereoperator.md) y [project](./projectoperator.md) en la tubería de entrada para reducir los datos a lo que le interesa.

Al buscar una fila interesante, puede profundizar aún más mediante la adición de sus valores específicos a su filtro `where` .

* Nota: diffpatterns tiene como objetivo encontrar patrones significativos (que capturan partes de la diferencia de datos entre los conjuntos) y no está destinado a las diferencias fila por fila.

**Ejemplo**

```kusto
StormEvents 
| where monthofyear(StartTime) == 5
| extend Damage = iff(DamageCrops + DamageProperty > 0 , 1 , 0)
| project State , EventType , Source , Damage, DamageCrops
| evaluate diffpatterns(Damage, "0", "1" )
```
|SegmentId|CountA|CountB|PercentA|PercentB|PercentDiffAB|State|EventType|Source|DamageCrops|
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

