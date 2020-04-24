---
title: operadores in y notin - Azure Data Explorer ? Microsoft Docs
description: En este artículo se describen los operadores in y notin en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/18/2019
ms.openlocfilehash: bd247de2bd211ae7be3da449e940899d2e8bb475
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513811"
---
# <a name="in-and-in-operators"></a>Operadores in e !in

Filtra un conjunto de registros en función del conjunto de valores proporcionado.

```kusto
Table1 | where col in ('value1', 'value2')
```

**Sintaxis**

*Sintaxis sensible a mayúsculas y minúsculas:*

*T* `|` T `where` *col* `in` col `(` *lista de expresiones escalares*`)`   
*T* `|` T `where` *col* `in` col `(` *expresión tabular*`)`   
 
*T* `|` T `where` *col* `!in` col `(` *lista de expresiones escalares*`)`  
*T* `|` T `where` *col* `!in` col `(` *expresión tabular*`)`   

*Sintaxis que no distingue mayúsculas de minúsculas:*

*T* `|` T `where` *col* `in~` col `(` *lista de expresiones escalares*`)`   
*T* `|` T `where` *col* `in~` col `(` *expresión tabular*`)`   
 
*T* `|` T `where` *col* `!in~` col `(` *lista de expresiones escalares*`)`  
*T* `|` T `where` *col* `!in~` col `(` *expresión tabular*`)`   

**Argumentos**

* *T* - La entrada tabular cuyos registros se van a filtrar.
* *col* - la columna para filtrar.
* *lista de expresiones* - una lista separada por comas de expresiones tabulares, escalares o literales  
* *expresión tabular* - una expresión tabular que tiene un conjunto de valores (en una expresión de mayúsculas y minúsculas tiene varias columnas, se utiliza la primera columna)

**Devuelve**

Filas en *T* para las que el predicado es`true`

**Notas**

* La lista de expresiones puede producir hasta `1,000,000` valores    
* Las matrices anidadas se aplanan en `x in (dynamic([1,[2,3]]))` una sola lista de valores, por ejemplo, se convierte en`x in (1,2,3)` 
* En el caso de expresiones tabulares, se selecciona la primera columna del conjunto de resultados   
* La adición de '' al operador `x in~ (expression)` hace `x !in~ (expression)`que los valores no distinten entre mayúsculas y minúsculas: o .

**Ejemplos:**  

**Un uso sencillo del operador 'in':**  

```kusto
StormEvents 
| where State in ("FLORIDA", "GEORGIA", "NEW YORK") 
| count
```

|Count|
|---|
|4775|  


**Un uso sencillo del operador 'in':**  

```kusto
StormEvents 
| where State in~ ("Florida", "Georgia", "New York") 
| count
```

|Count|
|---|
|4775|  

**Un uso sencillo del operador '!in':**  

```kusto
StormEvents 
| where State !in ("FLORIDA", "GEORGIA", "NEW YORK") 
| count
```

|Count|
|---|
|54291|  


**Uso de la matriz dinámica:**
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

La misma consulta se puede escribir como:

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

```kusto
let Lightning_By_State = materialize(StormEvents | summarize lightning_events = countif(EventType == 'Lightning') by State);
let Top_5_States = Lightning_By_State | top 5 by lightning_events | project State; 
Lightning_By_State
| extend State = iif(State in (Top_5_States), State, "Other")
| summarize sum(lightning_events) by State 
```

| State     | sum_lightning_events |
|-----------|----------------------|
| ALABAMA   | 29                   |
| Wisconsin | 31                   |
| TEXAS     | 55                   |
| Florida   | 85                   |
| Georgia   | 106                  |
| Otros     | 415                  |

**Uso de una lista estática devuelta por una función:**  

```kusto
StormEvents | where State in (InterestingStates()) | count

```

|Count|
|---|
|4775|  


Aquí está la definición de función:  

```kusto
.show function InterestingStates
```

|NOMBRE|Parámetros|Body|Carpeta|DocString|
|---|---|---|---|---|
|InterestingStates|()|• dynamic(["WASHINGTON", "FLORIDA", "GEORGIA", "NEW YORK"])
