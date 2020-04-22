---
title: 'Prácticas recomendadas de consultas: Explorador de azure Data Explorer Microsoft Docs'
description: En este artículo se describen los procedimientos recomendados de consulta en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/03/2020
ms.openlocfilehash: 3458c2fcd7fab3345f65393d7e9a13e844088a9f
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663868"
---
# <a name="query-best-practices"></a>Procedimientos recomendados sobre las consultas 

## <a name="general"></a>General

Hay varios "Dos y no saber" que puede seguir para que la consulta se ejecute más rápido.

### <a name="do"></a>Lo que es necesario hacer:

*   Usar primero filtros de tiempo. Kusto está altamente optimizado para utilizar filtros de tiempo.
*   Cuando se utilizan operadores de cadena:
    *   Prefiere `has` el `contains` operador antes de buscar tokens completos. `has`es más performante, ya que no tiene que buscar subcadenas.
    *   Prefiere el uso de operadores que distinguen mayúsculas de minúsculas cuando corresponda, ya que son más útiles. Por ejemplo, `==` prefiere `=~` `in` usar `in~`, `contains_cs` `contains` más y más `contains` / `contains_cs` (pero si `has` / `has_cs`se puede evitar por completo y utilizar, eso es aún mejor).
*   Prefiere buscar en una columna `*` específica en lugar de usar (búsqueda de texto completo en todas las columnas)
*   Si encuentra que la mayoría de las consultas tratan de extraer campos de [objetos dinámicos](./scalar-data-types/dynamic.md) en millones de filas, considere la posibilidad de materializar esta columna en el momento de la ingesta. De esta manera, solo pagará una vez por la extracción de columnas.  
*   Si tiene `let` una instrucción cuyo valor utiliza más de una vez, considere la posibilidad `materialize()`de utilizar la [función materialize()](./materializefunction.md) (consulte [algunas prácticas recomendadas](#materialize-function) sobre el uso ).

### <a name="dont"></a>Lo que debe evitar:

*   No intente nuevas consultas sin `limit [small number]` o `count` al final.
    La ejecución de consultas sin enlazar a través de un conjunto de datos desconocido puede producir que los rendimientos de los resultados se devuelvan al cliente, lo que da como resultado una respuesta lenta y el clúster está ocupado.
*   Si encuentra que está aplicando conversiones (JSON, cadena, etc.) a más de 1.000 millones de registros, cambie la forma de la consulta para reducir la cantidad de datos introducidos en la conversión.
*   No utilice `tolower(Col) == "lowercasestring"` para hacer comparaciones sin distinción entre mayúsculas y minúsculas. En su lugar, use `Col =~ "lowercasestring"`.
    *   Si los datos ya están en minúsculas (o en mayúsculas), `Col == "lowercasestring"` evite `Col == "UPPERCASESTRING"`usar comparaciones sin distinción entre mayúsculas y minúsculas y utilice (o ) en su lugar.
*   No filtre en una columna calculada si puede filtrar en una columna de tabla. En otras palabras: en lugar de `T | extend _value = <expression> | where predicate(_value)`, hacer: `T | where predicate(<expression>)`.

## <a name="summarize-operator"></a>Operador summarize

*   Cuando el grupo por claves del operador de resumen está con alta cardinalidad (mejores prácticas: por encima de 1 millón) se recomienda utilizar el [hint.strategy-shuffle](./shufflequery.md).

## <a name="join-operator"></a>Operador join

*   Cuando utilice el [operador join](./joinoperator.md), seleccione la tabla con menos filas para que sea la primera (más a la izquierda). 
*   Cuando utilice datos de operador de [combinación](./joinoperator.md) entre clústeres, ejecute la consulta en el lado "derecho" de la combinación (donde se encuentra la mayoría de los datos).
*   Cuando el lado izquierdo es pequeño (hasta 100.000 registros) y el lado derecho es grande, utilice [hint.strategy-broadcast](./broadcastjoin.md).
*   Cuando ambos lados de la unión son demasiado grandes y la clave de unión está con alta cardinalidad, utilice [hint.strategy-shuffle](./shufflequery.md).
    
## <a name="parse-operator-and-extract-function"></a>operador de análisis y función extract()

*   el operador de [análisis](./parseoperator.md) (modo simple) es útil cuando los valores de la columna de destino contienen cadenas que comparten el mismo formato o patrón.
Por ejemplo, para una `"Time = <time>, ResourceId = <resourceId>, Duration = <duration>, ...."`columna con valores como , al `parse` extraer los `extract()` valores de cada campo, utilice el operador en lugar de varias instrucciones.
*   [la función extract()](./extractfunction.md) es útil cuando las cadenas analizadas no siguen todos el mismo formato o patrón.
En tales casos, extraiga los valores necesarios mediante regEX.

## <a name="materialize-function"></a>función materialize()

*   Al utilizar la [función materialize(),](./materializefunction.md)intente insertar todos los operadores posibles que reduzcan el conjunto de datos materializado y mantengan la semántica de la consulta. Por ejemplo, los filtros o el proyecto solo requieren columnas.
    
    **Ejemplo**:

    ```kusto
    let materializedData = materialize(Table
    | where Timestamp > ago(1d));
    union (materializedData
    | where Text !has "somestring"
    | summarize dcount(Resource1)), (materializedData
    | where Text !has "somestring"
    | summarize dcount(Resource2))
    ```

* El filtro en Text es mutuo y se puede insertar en la expresión materialize.
    La consulta solo `Timestamp`necesita `Text` `Resource1` estas `Resource2` columnas , por lo que se recomienda proyectar estas columnas dentro de la expresión materializada.
    
    ```kusto
    let materializedData = materialize(Table
    | where Timestamp > ago(1d)
    | where Text !has "somestring"
    | project Timestamp, Resource1, Resource2, Text);
    union (materializedData
    | summarize dcount(Resource1)), (materializedData
    | summarize dcount(Resource2))
    ```
    
*   Si los filtros no son idénticos como en esta consulta:  

    ```kusto
    let materializedData = materialize(Table
    | where Timestamp > ago(1d));
    union (materializedData
    | where Text has "String1"
    | summarize dcount(Resource1)), (materializedData
    | where Text has "String2"
    | summarize dcount(Resource2))
    ```

*   Podría valer la pena (cuando el filtro combinado reduce drásticamente el resultado materializado) combinar `or` ambos filtros en el resultado materializado por una expresión lógica como en la consulta siguiente. Sin embargo, mantenga los filtros en cada tramo de unión para conservar la semántica de la consulta:
     
    ```kusto
    let materializedData = materialize(Table
    | where Timestamp > ago(1d)
    | where Text has "String1" or Text has "String2"
    | project Timestamp, Resource1, Resource2, Text);
    union (materializedData
    | where Text has "String1"
    | summarize dcount(Resource1)), (materializedData
    | where Text has "String2"
    | summarize dcount(Resource2))
    ```
    