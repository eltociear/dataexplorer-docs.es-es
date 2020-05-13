---
title: 'Instrucción Let: Explorador de datos de Azure'
description: En este artículo se describe la instrucción Let en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: c2e21f0b41f34b469e409109a2586f3e5fd98fa5
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271475"
---
# <a name="let-statement"></a>Instrucción Let

Permita que las instrucciones enlacen nombres a expresiones. Para el resto del ámbito en el que aparece la instrucción Let (ámbito global o en un ámbito de cuerpo de función), el nombre se puede usar para hacer referencia a su valor enlazado. Si ese nombre estaba enlazado previamente a otro valor, se usa el enlace de instrucción Let "más interno".

Las instrucciones Let mejoran la modularidad y la reutilización, ya que permiten dividir una expresión potencialmente compleja en varias partes, cada una enlazada a un nombre a través de la instrucción Let y componerlos juntos. También se pueden usar para crear funciones y vistas definidas por el usuario (expresiones en tablas cuyos resultados se parezcan a una tabla nueva).

Los nombres enlazados por instrucciones Let deben ser nombres de entidad válidos.

Las expresiones enlazadas por instrucciones Let pueden ser:
* De tipo escalar
* De tipo tabular
* Funciones definidas por el usuario (lambdas)

**Sintaxis**

`let`*Nombre* `=` de *ScalarExpression*  |  *TabularExpression*  |  *FunctionDefinitionExpression*

* *Name*: el nombre que se va a enlazar. El nombre debe ser un nombre de entidad válido y se permite el escape de nombre de entidad (por ejemplo, `["Name with spaces"]` ). 
* *ScalarExpression*: expresión con un resultado escalar cuyo valor se enlazará al nombre. Por ejemplo: `let one=1;`.
* *TabularExpression*: expresión con un resultado tabular cuyo valor se enlazará al nombre. Por ejemplo: `Logs | where Timestamp > ago(1h)`.
* *FunctionDefinitionExpression*: una expresión que produce una expresión lambda (una declaración de función anónima) que se va a enlazar al nombre.
  Por ejemplo: `let f=(a:int, b:string) { strcat(b, ":", a) }`.

Las expresiones lambda tienen la siguiente sintaxis:

[ `view` ] `(` [*TabularArguments*] [ `,` ] [*ScalarArguments*] `)` `{` *FunctionBody*`}`

`TabularArguments`-[*TabularArgName* `:` `(` [*AtrName* `:` *AtrType*] [ `,` ...] `)` ] [`,` ... ] [`,`]

 o:-[*TabularArgName* `:` `(` `*` `)` ]

`ScalarArguments`-[*ArgName* `:` *ArgType*] [ `,` ...]

* `view`solo puede aparecer en una expresión lambda sin parámetros (uno que no tiene argumentos) e indica que el nombre enlazado se incluirá cuando "todas las tablas" sean consultas (por ejemplo, al usar `union *` ).
* *TabularArguments* son la lista de argumentos de expresión tabular formal.
  Cada argumento tiene:
  * *TabularArgName* : el nombre del argumento tabular formal. El nombre puede aparecer en el *FunctionBody* y se enlaza a un valor determinado cuando se invoca la expresión lambda. 
  * Definición de esquema de tabla: lista de atributos con sus tipos (AtrName: AtrType).
  La expresión tabular que se usa en la invocación lambda debe tener todos estos atributos con los tipos coincidentes, pero no se limita a ellos. 
  ' (*) ' se puede usar como expresión tabular. En este caso, se puede usar cualquier expresión tabular en la invocación lambda y no se puede tener acceso a ninguna de sus columnas en la expresión lambda.
  Todos los argumentos tabulares deben aparecer antes que los argumentos escalares.
* *ScalarArguments* son la lista de argumentos escalares formales. 
  Cada argumento tiene:
  * *ArgName* : el nombre del argumento escalar formal. El nombre puede aparecer en el *FunctionBody* y se enlaza a un valor determinado cuando se invoca la expresión lambda.  
  * *ArgType* : el tipo del argumento escalar formal. Actualmente solo se admiten los siguientes tipos como un tipo de argumento lambda: `bool` , `string` , `long` , `datetime` , `timespan` , `real` y `dynamic` (y alias para estos tipos).

**Instrucciones Let múltiples y anidadas**

Se pueden utilizar varias instrucciones Let con `;` un delimitador entre ellas, tal y como se muestra en el ejemplo siguiente.
La última instrucción debe ser una expresión de consulta válida: 

```kusto
let start = ago(5h); 
let period = 2h; 
T | where Time > start and Time < start + period | ...
```

Se permiten las instrucciones Let anidadas y se pueden usar dentro de una expresión lambda.
Las instrucciones y los argumentos se pueden ver en el ámbito actual y interno del cuerpo de la función.

```kusto
let start_time = ago(5h); 
let end_time = start_time + 2h; 
T | where Time > start_time and Time < end_time | ...
```

**Ejemplos**

### <a name="using-let-to-define-constants"></a>Usar Let para definir constantes

En el ejemplo siguiente se enlaza el nombre `x` al literal escalar `1` y, a continuación, se usa en una instrucción de expresión tabular:

```kusto
let x = 1;
range y from x to x step x
```

Mismo ejemplo, pero en este caso, el nombre de la instrucción Let se proporciona mediante la `['name']` noción:

```kusto
let ['x'] = 1;
range y from x to x step x
```

Otro ejemplo que usa Let para los valores escalares:

```kusto
let n = 10;  // number
let place = "Dallas";  // string
let cutoff = ago(62d); // datetime
Events 
| where timestamp > cutoff 
    and city == place 
| take n
```

### <a name="using-multiple-let-statements"></a>Usar varias instrucciones Let

En el ejemplo siguiente se definen dos instrucciones Let donde una instrucción ( `foo2` ) utiliza otra ( `foo1` ).

```kusto
let foo1 = (_start:long, _end:long, _step:long) { range x from _start to _end step _step};
let foo2 = (_step:long) { foo1(1, 100, _step)};
foo2(2) | count
// Result: 50
```

### <a name="using-materialize-function"></a>Usar la función materializar

[`materialize`](materializefunction.md)la función permite almacenar en caché los resultados de la subconsulta durante el tiempo de ejecución de la consulta. 

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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
