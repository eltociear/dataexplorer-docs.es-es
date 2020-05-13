---
title: 'operadores in y notin: Azure Explorador de datos'
description: En este artículo se describen los operadores in y notin de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/18/2019
ms.openlocfilehash: cd11362c15e5ecfb80eab57b57b22f190f47da05
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271577"
---
# <a name="in-and-in-operators"></a>Operadores in e !in

Filtra un conjunto de registros basándose en el conjunto de valores proporcionado.

```kusto
Table1 | where col in ('value1', 'value2')
```

**Sintaxis**

*Sintaxis que distingue entre mayúsculas y minúsculas:*

*T* `|` `where` *col* `in` `(` *Lista de columnas de expresiones escalares*`)`   
*T* `|` `where` *col* `in` `(` *Expresión tabular* T col`)`   
 
*T* `|` `where` *col* `!in` `(` *Lista de columnas de expresiones escalares*`)`  
*T* `|` `where` *col* `!in` `(` *Expresión tabular* T col`)`   

*Sintaxis sin distinción entre mayúsculas y minúsculas:*

*T* `|` `where` *col* `in~` `(` *Lista de columnas de expresiones escalares*`)`   
*T* `|` `where` *col* `in~` `(` *Expresión tabular* T col`)`   
 
*T* `|` `where` *col* `!in~` `(` *Lista de columnas de expresiones escalares*`)`  
*T* `|` `where` *col* `!in~` `(` *Expresión tabular* T col`)`   

**Argumentos**

* *T* : la entrada tabular cuyos registros se van a filtrar.
* *col* : la columna que se va a filtrar.
* *lista de expresiones* : una lista separada por comas de expresiones tabulares, escalares o literales  
* *expresión tabular* : expresión tabular que tiene un conjunto de valores (en una expresión Case tiene varias columnas, se usa la primera columna)

**Devuelve**

Filas de *T* para las que el predicado es`true`

**Notas**

* La lista de expresiones puede generar valores de hasta `1,000,000`    
* Las matrices anidadas se acoplan en una sola lista de valores, por ejemplo, se `x in (dynamic([1,[2,3]]))` convierte en`x in (1,2,3)` 
* En el caso de las expresiones tabulares, se selecciona la primera columna del conjunto de resultados.   
* Al agregar ' ~ ' al operador, los valores no distinguen mayúsculas de minúsculas: `x in~ (expression)` o `x !in~ (expression)` .

**Ejemplos:**  

**Un uso sencillo del operador ' in ':**  

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where State in ("FLORIDA", "GEORGIA", "NEW YORK") 
| count
```

|Count|
|---|
|4775|  


**Un uso sencillo del operador ' in ~ ':**  

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where State in~ ("Florida", "Georgia", "New York") 
| count
```

|Count|
|---|
|4775|  

**Un uso sencillo del operador '! in ':**  

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents 
| where State !in ("FLORIDA", "GEORGIA", "NEW YORK") 
| count
```

|Count|
|---|
|54291|  


**Usar matriz dinámica:**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
let states = dynamic(['FLORIDA', 'ATLANTIC SOUTH', 'GEORGIA']);
StormEvents 
| where State in (states)
| count
```

|Count|
|---|
|3218|


**Un ejemplo de subconsulta:**  

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Using subquery
let Top_5_States = 
StormEvents
| summarize count() by State
| top 5 by count_; 
StormEvents 
| where State in (Top_5_States) 
| count
```

La misma consulta se puede escribir de la siguiente manera:

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Inline subquery 
StormEvents 
| where State in (
    ( StormEvents
    | summarize count() by State
    | top 5 by count_ )
) 
| count
```

|Count|
|---|
|14242|  

**Top con otro ejemplo:**  

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
let Lightning_By_State = materialize(StormEvents | summarize lightning_events = countif(EventType == 'Lightning') by State);
let Top_5_States = Lightning_By_State | top 5 by lightning_events | project State; 
Lightning_By_State
| extend State = iif(State in (Top_5_States), State, "Other")
| summarize sum(lightning_events) by State 
```

| Estado     | sum_lightning_events |
|-----------|----------------------|
| ALABAMA   | 29                   |
| WISCONSIN | 31                   |
| TEXAS     | 55                   |
| FLORIDA   | 85                   |
| Georgia   | 106                  |
| Otros     | 415                  |

**Usar una lista estática devuelta por una función:**  

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
StormEvents | where State in (InterestingStates()) | count

```

|Count|
|---|
|4775|  


Esta es la definición de función:  

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
.show function InterestingStates
```

|Nombre|Parámetros|Body|Carpeta|DocString|
|---|---|---|---|---|
|InterestingStates|()|{Dynamic (["WASHINGTON", "FLORIDA", "GEORGIA", "NUEVA YORK"])}
