---
title: Let (instrucción) - Explorador de azure Data Explorer (Explorador de datos de Azure) Microsoft Docs
description: En este artículo se describe la instrucción Let en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 890a6e21400048031e4ebd3df9749b6c47803e71
ms.sourcegitcommit: 653bfb3edf32553c52ef36b339c8b80713a601b0
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2020
ms.locfileid: "81524453"
---
# <a name="let-statement"></a>Instrucción Let

Las instrucciones Let enlazan nombres a expresiones. Para el resto del ámbito en el que aparece la instrucción let (ámbito global o en un ámbito de cuerpo de función), el nombre se puede utilizar para hacer referencia a su valor enlazado. Si ese nombre estaba enlazado previamente a otro valor, se utiliza el enlace de instrucción let "más interno".

Permitir que las instrucciones mejoren la modularidad y la reutilización, ya que permiten dividir una expresión potencialmente compleja en varias partes, cada una enlazada a un nombre a través de la instrucción let y componerlas juntas. También se pueden utilizar para crear funciones y vistas definidas por el usuario (expresiones sobre tablas cuyos resultados parecen una nueva tabla).

Los nombres enlazados por instrucciones let deben ser nombres de entidad válidos.

Las expresiones enlazadas por instrucciones let pueden ser:
* De tipo escalar
* De tipo tabular
* Funciones definidas por el usuario (lambdas)

**Sintaxis**

`let`*Nombre* `=` *ScalarExpression* | *TabularExpression* | *FunctionDefinitionExpression*

* *Nombre*: el nombre que se va a enlazar. El nombre debe ser un nombre de entidad válido y se `["Name with spaces"]`permite el escape del nombre de entidad (por ejemplo, ). 
* *ScalarExpression*: Una expresión con un resultado escalar cuyo valor se enlazará al nombre. Por ejemplo: `let one=1;`.
* *TabularExpression*: una expresión con un resultado tabular cuyo valor se enlazará al nombre. Por ejemplo: `Logs | where Timestamp > ago(1h)`.
* *FunctionDefinitionExpression*: expresión que produce una expresión lambda (una declaración de función anónima) que se va a enlazar al nombre.
  Por ejemplo: `let f=(a:int, b:string) { strcat(b, ":", a) }`.

Las expresiones Lambda tienen la sintaxis siguiente:

[`view` `(`] [*TabularArguments*`,`][ ][`)` `{` *ScalarArguments*] *FunctionBody*`}`

`TabularArguments`- [*TabularArgName* `:` `(`[*AtrName* `:` `,` *AtrType*] [ ... ] `)`] [`,` ... ] [`,`]

 o: - [*TabularArgName* `:` `(` `*` `)`]

`ScalarArguments`- [*ArgName* `:` *ArgType*] [`,` ... ]

* `view`puede aparecer solo en una lambda sin parámetros (una que no tiene argumentos) e indica que el nombre `union *`enlazado se incluirá cuando "todas las tablas" son consultas (por ejemplo, cuando se usa ).
* *TabularArguments* es la lista de los argumentos de expresión tabular formal.
  Cada argumento tiene:
  * *TabularArgName* - el nombre del argumento tabular formal. El nombre, a continuación, puede aparecer en el *FunctionBody* y está enlazado a un valor determinado cuando se invoca la expresión lambda. 
  * Definición de esquema de tabla - una lista de atributos con sus tipos (AtrName : AtrType).
  La expresión tabular que se utiliza en la invocación lambda debe tener todos estos atributos con los tipos coincidentes, pero no se limita a ellos. 
  '(*)' se puede utilizar como expresión tabular. En este caso, se puede utilizar cualquier expresión tabular en la invocación lambda y no se puede tener acceso a ninguna de sus columnas en la expresión lambda.
  Todos los argumentos tabulares deben aparecer antes de los argumentos escalares.
* *ScalarArguments* es la lista de los argumentos escalares formales. 
  Cada argumento tiene:
  * *ArgName* - el nombre del argumento escalar formal. El nombre, a continuación, puede aparecer en el *FunctionBody* y está enlazado a un valor determinado cuando se invoca la expresión lambda.  
  * *ArgType* - el tipo del argumento escalar formal. Actualmente, solo se admiten los `bool`siguientes `string` `long`tipos `datetime` `timespan`como `real`un `dynamic` tipo de argumento lambda: , , , , , , y (y alias para estos tipos).

**Declaraciones de let múltiples y anidadas**

Se pueden utilizar varias `;` instrucciones let con delimitador entre ellas, como se muestra en el ejemplo siguiente.
La última instrucción debe ser una expresión de consulta válida: 

```kusto
let start = ago(5h); 
let period = 2h; 
T | where Time > start and Time < start + period | ...
```

Las instrucciones let anidadas se permiten y se pueden usar dentro de una expresión lambda.
Las instrucciones Let y Arguments están visibles en el ámbito actual e interno del cuerpo de la función.

```kusto
let start_time = ago(5h); 
let end_time = start_time + 2h; 
T | where Time > start_time and Time < end_time | ...
```

**Ejemplos**

### <a name="using-let-to-define-constants"></a>Uso de let para definir constantes

En el ejemplo siguiente `x` se enlaza el `1`nombre al literal escalar y, a continuación, se utiliza en una instrucción de expresión tabular:

```kusto
let x = 1;
range y from x to x step x
```

El mismo ejemplo, pero en este caso - `['name']` el nombre de la instrucción let se da usando la noción:

```kusto
let ['x'] = 1;
range y from x to x step x
```

Otro ejemplo que utiliza let para los valores escalares:

```kusto
let n = 10;  // number
let place = "Dallas";  // string
let cutoff = ago(62d); // datetime
Events 
| where timestamp > cutoff 
    and city == place 
| take n
```

### <a name="using-multiple-let-statements"></a>Uso de varias instrucciones let

En el ejemplo siguiente se definen`foo2`dos instrucciones`foo1`let en las que una instrucción ( ) utiliza otra ( ).

```kusto
let foo1 = (_start:long, _end:long, _step:long) { range x from _start to _end step _step};
let foo2 = (_step:long) { foo1(1, 100, _step)};
foo2(2) | count
// Result: 50
```

### <a name="using-materialize-function"></a>Uso de la función de materialización

[`materialize`](materializefunction.md)función permite almacenar en caché los resultados de las subconsultas durante el tiempo de ejecución de la consulta. 

```kusto
let totalPagesPerDay = PageViews
| summarize by Page, Day = startofday(Timestamp)
| summarize count() by Day;
let materializedScope = PageViews
| summarize by Page, Day = startofday(Timestamp);
let cachedResult = materialize(materializedScope);
cachedResult
| project Page, Day1 = Day
| join kind = inner
(
    cachedResult
    | project Page, Day2 = Day
)
on Page
| where Day2 > Day1
| summarize count() by Day1, Day2
| join kind = inner
    totalPagesPerDay
on $left.Day1 == $right.Day
| project Day1, Day2, Percentage = count_*100.0/count_1


```

|Día 1|Día 2|Porcentaje|
|---|---|---|
|2016-05-01 00:00:00.0000000|2016-05-02 00:00:00.0000000|34.0645725975255|
|2016-05-01 00:00:00.0000000|2016-05-03 00:00:00.0000000|16.618368960101|
|2016-05-02 00:00:00.0000000|2016-05-03 00:00:00.0000000|14.6291376489636|
