---
title: 'operador de combinación: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe el operador join en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 86d883c8d0214d278099260fa2406b91b776b4d1
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765818"
---
# <a name="join-operator"></a>Operador join

combina las filas de dos tablas para formar una nueva tabla haciendo coincidir los valores de las columnas especificadas de cada tabla.

```kusto
Table1 | join (Table2) on CommonColumn, $left.Col1 == $right.Col2
```

**Sintaxis**

*LeftTable* `|` `join` [*JoinParameters*] `(` *Atributos* *rightTable* `)` `on`

**Argumentos**

* *LeftTable*: La tabla **izquierda** o expresión tabular (a veces denominada tabla **externa)** cuyas filas se van a combinar. Denotado `$left`como .

* *RightTable*: La tabla **derecha** o expresión tabular (a veces denominada * tabla*interna)* cuyas filas se van a combinar. Denotado `$right`como .

* *Atributos*: Una o más reglas (separadas por comas) que describen cómo las filas de *LeftTable* coinciden con las filas de *RightTable*. Se evalúan varias `and` reglas mediante el operador lógico.
  Una regla puede ser una de las:

  |Tipo de regla        |Sintaxis                                          |Predicate                                                      |
  |-----------------|------------------------------------------------|---------------------------------------------------------------|
  |Igualdad por nombre |*ColumnName*                                    |`where`*LeftTable*. *ColumnName* `==` *RightTable*. *ColumnName*|
  |Igualdad por valor|`$left.`*LeftColumn* `==` `$right.` *RightColumn*|`where``$left.` *LeftColumn* `==` LeftColumn `$right.` *RightColumn*       |

> [!NOTE]
> En caso de "igualdad por valor", los nombres de columna *deben* calificarse con la tabla de propietario aplicable denotada por `$left` y `$right` notaciones.

* *JoinParameters*: Cero o más parámetros (separados por espacios) en forma de *valor* de *nombre* `=` que controlan el comportamiento de la operación de coincidencia de filas y el plan de ejecución. Se admiten los siguientes parámetros: 

::: zone pivot="azuredataexplorer"

  |Nombre           |Valores                                        |Descripción                                  |
  |---------------|----------------------------------------------|---------------------------------------------|
  |`kind`         |Tipos de combinación|Ver [Unirse a sabores](#join-flavors)|                                             |
  |`hint.remote`  |`auto`, `left`, `local`, `right`              |Consulte [Unión entre clústeres](joincrosscluster.md)|
  |`hint.strategy`|Sugerencias de ejecución                               |Consulte [Sugerencias de unión](#join-hints)                |

::: zone-end

::: zone pivot="azuremonitor"

  |Nombre           |Valores                                        |Descripción                                  |
  |---------------|----------------------------------------------|---------------------------------------------|
  |`kind`         |Tipos de combinación|Ver [Unirse a sabores](#join-flavors)|                                             |
  |`hint.remote`  |`auto`, `left`, `local`, `right`              |                                             |
  |`hint.strategy`|Sugerencias de ejecución                               |Consulte [Sugerencias de unión](#join-hints)                |

::: zone-end


> [!WARNING]
> El sabor de `kind` unión predeterminado, `innerunique`si no se especifica, es . Esto es diferente de algunos otros `inner` productos de análisis, que tienen como sabor predeterminado. Lea atentamente a [continuación](#join-flavors) para comprender las diferencias entre los diferentes tipos y asegurarse de que la consulta produce los resultados previstos.

**Devuelve**

El esquema de salida depende del sabor de la unión:

 * `kind=leftanti`, `kind=leftsemi`:

     La tabla de resultados contiene columnas del lado izquierdo solamente.

     
 * `kind=rightanti`, `kind=rightsemi`:

     La tabla de resultados contiene columnas del lado derecho solamente.

     
*  `kind=innerunique`, `kind=inner`, `kind=leftouter`, `kind=rightouter`, `kind=fullouter`

     Una columna por cada columna en cada una de las dos tablas, incluidas las claves coincidentes. Si hay conflictos de nombres las columnas de la derecha se cambiarán automáticamente de nombre.

     
Los registros de salida dependen del sabor de la unión:

 * `kind=leftanti`, `kind=leftantisemi`

     Devuelve todos los registros del lado izquierdo que no tienen coincidencias desde la derecha.     
     
 * `kind=rightanti`, `kind=rightantisemi`

     Devuelve todos los registros del lado derecho que no tienen coincidencias desde la izquierda.  
      
*  `kind=innerunique`, `kind=inner`, `kind=leftouter`, `kind=rightouter`, `kind=fullouter`, `kind=leftsemi`, `kind=rightsemi`

    Una fila por cada correspondencia entre las tablas de entrada. Una coincidencia es una fila seleccionada de una tabla `on` que tiene el mismo valor para todos los campos que una fila de la otra tabla con estas restricciones:

   - `kind`Indeterminado`kind=innerunique`

    Solo se compara una fila del lado izquierdo para cada valor de la clave `on` . La salida contiene una fila por cada coincidencia de esta fila con filas de la derecha.
    
   - `kind=leftsemi`
   
    Devuelve todos los registros del lado izquierdo que tienen coincidencias desde la derecha.
    
   - `kind=rightsemi`
   
       Devuelve todos los registros del lado derecho que tienen coincidencias desde la izquierda.

   - `kind=inner`
 
    Hay una fila en la salida por cada combinación de filas coincidentes de la izquierda y la derecha.

   - `kind=leftouter` (o `kind=rightouter` o `kind=fullouter`)

    Además de las coincidencias internas, hay una fila por cada fila de la izquierda (o derecha), incluso si no tiene ninguna coincidencia. En este caso, las celdas de salida sin coincidencias contienen valores NULL.
    Si hay varias filas con los mismos valores para esos campos, obtendrá filas para todas las combinaciones.

 

**Sugerencias**

Para obtener el mejor rendimiento:

* Si una tabla siempre es menor que la otro, úsela como lado izquierdo (canalizado) de la combinación.

**Ejemplo**

Obtener más actividades de un registro en el que algunas entradas marcan el inicio y fin de una actividad. 

```kusto
let Events = MyLogTable | where type=="Event" ;
Events
| where Name == "Start"
| project Name, City, ActivityId, StartTime=timestamp
| join (Events
    | where Name == "Stop"
        | project StopTime=timestamp, ActivityId)
    on ActivityId
| project City, ActivityId, StartTime, StopTime, Duration = StopTime - StartTime
```

```kusto
let Events = MyLogTable | where type=="Event" ;
Events
| where Name == "Start"
| project Name, City, ActivityIdLeft = ActivityId, StartTime=timestamp
| join (Events
        | where Name == "Stop"
        | project StopTime=timestamp, ActivityIdRight = ActivityId)
    on $left.ActivityIdLeft == $right.ActivityIdRight
| project City, ActivityId, StartTime, StopTime, Duration = StopTime - StartTime
```

[Más información sobre este ejemplo](./samples.md#activities).

## <a name="join-flavors"></a>Tipos de combinación

El tipo exacto del operador de unión se especifica con la palabra clave de variante. A partir de hoy, Kusto soporta los siguientes sabores del operador de unión: 

|Tipo de combinación|Descripción|
|--|--|
|[`innerunique`](#default-join-flavor)(o vacío por defecto)|Unión interna con deduplicación del lado izquierdo|
|[`inner`](#inner-join)|Unión interna estándar|
|[`leftouter`](#left-outer-join)|Combinación externa izquierda|
|[`rightouter`](#right-outer-join)|Unión exterior derecha|
|[`fullouter`](#full-outer-join)|Unión exterior completa|
|[`leftanti`](#left-anti-join), [`anti`](#left-anti-join) o[`leftantisemi`](#left-anti-join)|Izquierda anti unión|
|[`rightanti`](#right-anti-join)O[`rightantisemi`](#right-anti-join)|Derecha anti unión|
|[`leftsemi`](#left-semi-join)|Semi unión a la izquierda|
|[`rightsemi`](#right-semi-join)|Semi unión derecha|

### <a name="inner-and-innerunique-join-operator-flavors"></a>sabores de operador de unión interior e innerúnico

-    Cuando se usa el sabor de **combinación interna,** habrá una fila en la salida para cada combinación de filas coincidentes de izquierda y derecha sin deduplicaciones de teclas izquierdas. La salida será un producto cartesiano de teclas izquierda y derecha.
    Ejemplo de **unión interna:**

```kusto
let t1 = datatable(key:long, value:string)  
[
1, "val1.1",  
1, "val1.2"  
];
let t2 = datatable(key:long, value:string)  
[  
1, "val1.3", 
1, "val1.4"  
];
t1
| join kind = inner
    t2
on key
```

|key|value|key1|value1|
|---|---|---|---|
|1|val1.2|1|val1.3|
|1|val1.1|1|val1.3|
|1|val1.2|1|val1.4|
|1|val1.1|1|val1.4|

-    El uso de sabor de **unión interior único** deduplicará las teclas del lado izquierdo y habrá una fila en la salida de cada combinación de teclas izquierda deduplicadas y teclas derechas.
    Ejemplo de **combinación innerunique** para los mismos conjuntos de datos utilizados anteriormente, Tenga en cuenta que **el sabor interior único** para este caso puede producir dos salidas posibles y ambos son correctos.
    En la primera salida, el operador join escogió aleatoriamente la primera clave que aparece en t1 con el valor "val1.1" y la emparedó con las teclas t2 mientras que en la segunda, el operador join escogió aleatoriamente la segunda clave aparece en t1 que tiene el valor "val1.2" y la coincide con las teclas t2:

```kusto
let t1 = datatable(key:long, value:string)  
[
1, "val1.1",  
1, "val1.2"  
];
let t2 = datatable(key:long, value:string)  
[  
1, "val1.3", 
1, "val1.4"  
];
t1
| join kind = innerunique
    t2
on key
```

|key|value|key1|value1|
|---|---|---|---|
|1|val1.1|1|val1.3|
|1|val1.1|1|val1.4|

```kusto
let t1 = datatable(key:long, value:string)  
[
1, "val1.1",  
1, "val1.2"  
];
let t2 = datatable(key:long, value:string)  
[  
1, "val1.3", 
1, "val1.4"  
];
t1
| join kind = innerunique
    t2
on key
```

|key|value|key1|value1|
|---|---|---|---|
|1|val1.2|1|val1.3|
|1|val1.2|1|val1.4|


-    Kusto está optimizado para empujar `join` los filtros que vienen después del lado de unión adecuado a la izquierda o a la derecha cuando sea posible.
    A veces, cuando el sabor utilizado es **innerunique** y el filtro se puede propagar al lado izquierdo de la unión, entonces se propagará automáticamente y las teclas que se aplican a ese filtro siempre aparecerán en la salida.
    por ejemplo, el uso del ` where value == "val1.2" ` ejemplo anterior y la adición de filtro siempre dará el segundo resultado y nunca dará el primer resultado para los conjuntos de datos utilizados:

```kusto
let t1 = datatable(key:long, value:string)  
[
1, "val1.1",  
1, "val1.2"  
];
let t2 = datatable(key:long, value:string)  
[  
1, "val1.3", 
1, "val1.4"  
];
t1
| join kind = innerunique
    t2
on key
| where value == "val1.2"
```

|key|value|key1|value1|
|---|---|---|---|
|1|val1.2|1|val1.3|
|1|val1.2|1|val1.4|
 
### <a name="default-join-flavor"></a>Sabor de unión por defecto
    
    X | join Y on Key
    X | join kind=innerunique Y on Key
     
Vamos a usar dos tablas de ejemplo para explicar el funcionamiento de la combinación: 
 
Tabla X 

|Clave |Valor1 
|---|---
|a |1 
|b |2 
|b |3 
|c |4 

Tabla Y 

|Clave |Valor2 
|---|---
|b |10 
|c |20 
|c |30 
|d |40 
 
La combinación predeterminada realiza una combinación interna después de la desduplicación en el lado izquierdo de la clave de combinación (la desduplicación conserva el primer registro). Con esta instrucción: 

    X | join Y on Key 

el lado izquierdo efectivo de la combinación (la tabla X después de la desduplicación) sería: 

|Clave |Valor1 
|---|---
|a |1 
|b |2 
|c |4 

y el resultado de la combinación sería: 

```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join Y on Key
```

|Clave|Valor1|Tecla1|Valor2|
|---|---|---|---|
|b|2|b|10|
|c|4|c|20|
|c|4|c|30|


(Tenga en cuenta que las teclas 'a' y 'd' no aparecen en la salida, ya que no había teclas coincidentes en ambos lados izquierdo y derecho). 
 
(Históricamente, esta fue la primera implementación de la unión admitida por la versión inicial de Kusto; es útil en los escenarios típicos de análisis de registro/rastreo donde queremos correlacionar dos eventos (cada uno que coincida con algún criterio de filtrado) bajo el mismo identificador de correlación, y recuperar todas las apariencias del fenómeno que estamos buscando, ignorando múltiples apariciones de los registros de seguimiento que contribuyen.)
 
### <a name="inner-join"></a>Combinación interna

Se trata de la combinación interna estándar tal y como se conoce en el mundo SQL. Se genera un registro de salida cada vez que un registro del lado izquierdo tenga la misma clave de combinación que el registro del lado derecho. 
 
```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=inner Y on Key
```

|Clave|Valor1|Tecla1|Valor2|
|---|---|---|---|
|b|3|b|10|
|b|2|b|10|
|c|4|c|20|
|c|4|c|30|

Observe que (b,10), procedente del lado derecho, se ha combinado dos veces: con (b,2) y (b, 3) a la izquierda; además, (c,4), procedente del lado izquierdo, se ha combinado dos veces: con (c,20) y (c,30) en el lado derecho. 

### <a name="left-outer-join"></a>Combinación externa izquierda 

El resultado de una combinación externa izquierda para las tablas X e Y contiene siempre todos los registros de la tabla izquierda (X), incluso si la condición de combinación no encuentra ningún registro coincidente en la tabla derecha (Y). 
 
```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=leftouter Y on Key
```

|Clave|Valor1|Tecla1|Valor2|
|---|---|---|---|
|b|3|b|10|
|b|2|b|10|
|c|4|c|20|
|c|4|c|30|
|a|1|||

 
### <a name="right-outer-join"></a>Unión exterior derecha 

Es similar a la combinación externa izquierda, pero se invierte el tratamiento de las tablas. 
 
```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=rightouter Y on Key
```

|Clave|Valor1|Tecla1|Valor2|
|---|---|---|---|
|b|3|b|10|
|b|2|b|10|
|c|4|c|20|
|c|4|c|30|
|||d|40|

 
### <a name="full-outer-join"></a>Unión exterior completa 

Conceptualmente, una combinación externa completa combina el efecto de aplicar combinaciones derecha e izquierda externas. Cuando los registros de las tablas unidas no coinciden, el conjunto de resultados tendrá valores NULL para cada columna de la tabla que carece de una fila coincidente. Para aquellos registros que coincidan, se producirá una única fila del conjunto de resultados (que contiene campos completados con valores de ambas tablas). 
 
```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=fullouter Y on Key
```

|Clave|Valor1|Tecla1|Valor2|
|---|---|---|---|
|b|3|b|10|
|b|2|b|10|
|c|4|c|20|
|c|4|c|30|
|||d|40|
|a|1|||

 
### <a name="left-anti-join"></a>Izquierda anti unión

Anti join izquierdo devuelve todos los registros del lado izquierdo que no coinciden con ningún registro del lado derecho. 
 
```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=leftanti Y on Key
```

|Clave|Valor1|
|---|---|
|a|1|

La anticombinación modela la consulta "NOT IN". 

### <a name="right-anti-join"></a>Derecha anti unión

Anti join derecho devuelve todos los registros del lado derecho que no coinciden con ningún registro del lado izquierdo. 
 
```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=rightanti Y on Key
```

|Clave|Valor2|
|---|---|
|d|40|

La anticombinación modela la consulta "NOT IN". 

### <a name="left-semi-join"></a>Semi unión a la izquierda

La semi combinación izquierda devuelve todos los registros del lado izquierdo que coinciden con un registro del lado derecho. Solo se devuelven las columnas del lado izquierdo. 

```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=leftsemi Y on Key
```

|Clave|Valor1|
|---|---|
|b|3|
|b|2|
|c|4|

### <a name="right-semi-join"></a>Semi unión derecha

La semi combinación derecha devuelve todos los registros del lado derecho que coinciden con un registro del lado izquierdo. Solo se devuelven las columnas del lado derecho. 

```kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X | join kind=rightsemi Y on Key
```

|Clave|Valor2|
|---|---|
|b|10|
|c|20|
|c|30|


### <a name="cross-join"></a>Unirse cruzadas

Kusto no proporciona de forma nativa un sabor de unión cruzada (es `kind=cross`decir, no se puede marcar el operador con ).
Sin embargo, no es difícil simular esto al presentar una clave ficticia:

    X | extend dummy=1 | join kind=inner (Y | extend dummy=1) on dummy

## <a name="join-hints"></a>Unir sugerencias

El `join` operador admite una serie de sugerencias que controlan la forma en que se ejecuta una consulta.
Estos no cambian la `join`semántica de , pero pueden afectar a su rendimiento.

Sugerencias de unión explicadas en los siguientes artículos: 
* `hint.shufflekey=<key>`y `hint.strategy=shuffle`  -  [consulta aleatoria](shufflequery.md)
* `hint.strategy=broadcast` - [difusión unirse](broadcastjoin.md)
* `hint.remote=<strategy>` - [unión entre clústeres](joincrosscluster.md)
