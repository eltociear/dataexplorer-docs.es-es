---
title: 'operador de combinación: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe el operador de combinación en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 2961f27226175fe81b7d9c82a6366b36134d805f
ms.sourcegitcommit: 31af2dfa75b5a2f59113611cf6faba0b45d29eb5
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/05/2020
ms.locfileid: "84454123"
---
# <a name="join-operator"></a>Operador join

combina las filas de dos tablas para formar una nueva tabla haciendo coincidir los valores de las columnas especificadas de cada tabla.

```kusto
Table1 | join (Table2) on CommonColumn, $left.Col1 == $right.Col2
```

**Sintaxis**

*LeftTable* `|` `join`Atributos*JoinParameters*de `(` *RightTable* `)` `on` *Attributes* [JoinParameters]

**Argumentos**

* *LeftTable*: la tabla **izquierda** o la expresión tabular (a veces denominada tabla **externa** ) cuyas filas se van a combinar. Se indica como `$left` .

* *RightTable*: la tabla o expresión tabular **derecha** (a veces denominada * tabla*interna* ) cuyas filas se van a combinar. Se indica como `$right` .

* *Atributos*: una o más reglas (separadas por comas) que describen cómo las filas de *LeftTable* coinciden con las filas de *RightTable*. Varias reglas se evalúan mediante el `and` operador lógico.
  Una regla puede ser una de las siguientes:

  |Tipo de regla        |Sintaxis                                          |Predicate                                                      |
  |-----------------|------------------------------------------------|---------------------------------------------------------------|
  |Igualdad por nombre |*ColumnName*                                    |`where`*LeftTable*. *ColumnName* `==` *RightTable*. *ColumnName*|
  |Igualdad por valor|`$left.`*LeftColumn* `==` `$right.` *RightColumn*|`where``$left.` *LeftColumn* `==` LeftColumn `$right.` *RightColumn*       |

> [!NOTE]
> En el caso de ' igualdad por valor ', los nombres de columna se *deben* calificar con la tabla propietaria aplicable indicada por las `$left` `$right` notaciones y.

* *JoinParameters*: cero o más parámetros (separados por espacios) en forma de valor de *nombre* `=` *Value* que controlan el comportamiento de la operación de coincidencia de filas y el plan de ejecución. Se admiten los siguientes parámetros: 

::: zone pivot="azuredataexplorer"

  |Nombre           |Valores                                        |Descripción                                  |
  |---------------|----------------------------------------------|---------------------------------------------|
  |`kind`         |Tipos de combinación|Vea [tipos de combinación](#join-flavors)|                                             |
  |`hint.remote`  |`auto`, `left`, `local`, `right`              |Consulte [combinación entre clústeres](joincrosscluster.md)|
  |`hint.strategy`|Sugerencias de ejecución                               |Vea [sugerencias de combinación](#join-hints)                |

::: zone-end

::: zone pivot="azuremonitor"

  |Nombre           |Valores                                        |Descripción                                  |
  |---------------|----------------------------------------------|---------------------------------------------|
  |`kind`         |Tipos de combinación|Vea [tipos de combinación](#join-flavors)|                                             |
  |`hint.remote`  |`auto`, `left`, `local`, `right`              |                                             |
  |`hint.strategy`|Sugerencias de ejecución                               |Vea [sugerencias de combinación](#join-hints)                |

::: zone-end


> [!WARNING]
> El tipo de combinación predeterminado, si `kind` no se especifica, es `innerunique` . Esto es diferente de otros productos de análisis, que tienen `inner` como el tipo predeterminado. Lea detenidamente a [continuación](#join-flavors) para comprender las diferencias entre los distintos tipos y asegurarse de que la consulta produce los resultados deseados.

**Devuelve**

El esquema de salida depende del tipo de combinación:

 * `kind=leftanti`, `kind=leftsemi`:

     La tabla de resultados solo contiene las columnas del lado izquierdo.

     
 * `kind=rightanti`, `kind=rightsemi`:

     La tabla de resultados solo contiene las columnas del lado derecho.

     
*  `kind=innerunique`, `kind=inner`, `kind=leftouter`, `kind=rightouter`, `kind=fullouter`

     Una columna por cada columna en cada una de las dos tablas, incluidas las claves coincidentes. Si hay conflictos de nombres las columnas de la derecha se cambiarán automáticamente de nombre.

     
Los registros de salida dependen del tipo de combinación:

 * `kind=leftanti`, `kind=leftantisemi`

     Devuelve todos los registros del lado izquierdo que no tienen coincidencias de la derecha.     
     
 * `kind=rightanti`, `kind=rightantisemi`

     Devuelve todos los registros del lado derecho que no tienen coincidencias de la izquierda.  
      
*  `kind=innerunique`, `kind=inner`, `kind=leftouter`, `kind=rightouter`, `kind=fullouter`, `kind=leftsemi`, `kind=rightsemi`

    Una fila por cada correspondencia entre las tablas de entrada. Una coincidencia es una fila seleccionada de una tabla que tiene el mismo valor para todos los `on` campos que una fila de la otra tabla con estas restricciones:

   - `kind`no especificado`kind=innerunique`

    Solo se compara una fila del lado izquierdo para cada valor de la clave `on` . La salida contiene una fila por cada coincidencia de esta fila con filas de la derecha.
    
   - `kind=leftsemi`
   
    Devuelve todos los registros del lado izquierdo que tienen coincidencias de la derecha.
    
   - `kind=rightsemi`
   
       Devuelve todos los registros del lado derecho que tienen coincidencias de la izquierda.

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

[Más información sobre este ejemplo](./samples.md#get-sessions-from-start-and-stop-events).

## <a name="join-flavors"></a>Tipos de combinación

El tipo exacto del operador de unión se especifica con la palabra clave de variante. A partir de hoy, Kusto admite los siguientes tipos de operadores de combinación: 

|Tipo de combinación|Descripción|
|--|--|
|[`innerunique`](#default-join-flavor)(o vacío como predeterminado)|Combinación interna con desduplicación del lado izquierdo|
|[`inner`](#inner-join)|Combinación interna estándar|
|[`leftouter`](#left-outer-join)|Combinación externa izquierda|
|[`rightouter`](#right-outer-join)|Combinación externa derecha|
|[`fullouter`](#full-outer-join)|Combinación externa completa|
|[`leftanti`](#left-anti-join), [`anti`](#left-anti-join) o[`leftantisemi`](#left-anti-join)|Anticombinación izquierda|
|[`rightanti`](#right-anti-join)de[`rightantisemi`](#right-anti-join)|Anti-combinación derecha|
|[`leftsemi`](#left-semi-join)|Semicombinación izquierda|
|[`rightsemi`](#right-semi-join)|Semicombinación derecha|

### <a name="inner-and-innerunique-join-operator-flavors"></a>tipos de operadores Inner y innerunique join

-    Cuando se usa el **tipo de combinación interna**, habrá una fila en la salida para cada combinación de filas coincidentes de izquierda y derecha sin desduplicar las claves de la izquierda. La salida será un producto cartesiano de las claves izquierda y derecha.
    Ejemplo de **combinación interna**:

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
|1|Val 1.2|1|Val 1.3|
|1|Val 1.1|1|Val 1.3|
|1|Val 1.2|1|Val 1.4|
|1|Val 1.1|1|Val 1.4|

-    Al usar **innerunique join Flavor** , se desduplicarán las claves del lado izquierdo y habrá una fila en la salida de cada combinación de claves izquierda desduplicadas y claves correctas.
    Ejemplo de **innerunique join** para los mismos conjuntos de valores usados anteriormente, tenga en cuenta que en este caso **innerunique Flavor** puede producir dos salidas posibles y ambas son correctas.
    En la primera salida, el operador de combinación eligió aleatoriamente la primera clave que aparece en T1 con el valor "Val 1.1" y la coincidía con las claves T2 en el segundo, el operador de combinación que selecciona aleatoriamente la segunda clave aparece en T1, que tiene el valor "Val 1.2" y la coincidía con las claves T2:

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
|1|Val 1.1|1|Val 1.3|
|1|Val 1.1|1|Val 1.4|

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
|1|Val 1.2|1|Val 1.3|
|1|Val 1.2|1|Val 1.4|


-    Kusto está optimizado para los filtros de extracción que se encuentran después de `join` hacia la combinación adecuada de izquierda o derecha cuando sea posible.
    A veces, cuando el tipo utilizado es **innerunique** y el filtro se puede propagar al lado izquierdo de la combinación, se propagará automáticamente y las claves que se apliquen a ese filtro aparecerán siempre en la salida.
    por ejemplo, si se usa el ejemplo anterior y se agrega el filtro ` where value == "val1.2" ` , siempre se dará el segundo resultado y nunca dará el primer resultado para los conjuntos de valores usados:

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
|1|Val 1.2|1|Val 1.3|
|1|Val 1.2|1|Val 1.4|
 
### <a name="default-join-flavor"></a>Tipo de combinación predeterminado
    
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


(Tenga en cuenta que las claves ' a ' y ' d ' no aparecen en la salida, ya que no había claves coincidentes en los lados izquierdo y derecho). 
 
(Históricamente, se trata de la primera implementación de la combinación admitida por la versión inicial de Kusto; es útil en los escenarios de análisis de registro/seguimiento típicos en los que queremos poner en correlación dos eventos (cada uno de los cuales coincide con algún criterio de filtrado) bajo el mismo identificador de correlación y obtener todos los aspectos del fenómeno que estamos buscando, omitiendo varias apariencias de los registros de
 
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

 
### <a name="right-outer-join"></a>Combinación externa derecha 

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

 
### <a name="full-outer-join"></a>Combinación externa completa 

Conceptualmente, una combinación externa completa combina el efecto de aplicar combinaciones derecha e izquierda externas. Cuando los registros de las tablas combinadas no coinciden, el conjunto de resultados tendrá valores NULL para todas las columnas de la tabla que carezcan de una fila coincidente. Para aquellos registros que coincidan, se producirá una única fila del conjunto de resultados (que contiene campos completados con valores de ambas tablas). 
 
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

 
### <a name="left-anti-join"></a>Anticombinación izquierda

La anticombinación izquierda devuelve todos los registros del lado izquierdo que no coinciden con ningún registro del lado derecho. 
 
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

### <a name="right-anti-join"></a>Anti-combinación derecha

La anticombinación derecha devuelve todos los registros del lado derecho que no coinciden con ningún registro del lado izquierdo. 
 
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

### <a name="left-semi-join"></a>Semicombinación izquierda

Left semi join devuelve todos los registros del lado izquierdo que coinciden con un registro del lado derecho. Solo se devuelven las columnas del lado izquierdo. 

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

### <a name="right-semi-join"></a>Semicombinación derecha

Right semi join devuelve todos los registros del lado derecho que coinciden con un registro del lado izquierdo. Solo se devuelven las columnas del lado derecho. 

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


### <a name="cross-join"></a>Combinación cruzada

Kusto no proporciona de forma nativa un tipo de combinación cruzada (es decir, no se puede marcar el operador con `kind=cross` ).
Sin embargo, no es difícil simular esto con una clave ficticia:

    X | extend dummy=1 | join kind=inner (Y | extend dummy=1) on dummy

## <a name="join-hints"></a>Sugerencias de combinación

El `join` operador admite varias sugerencias que controlan el modo en que se ejecuta una consulta.
Estos no cambian la semántica de `join` , pero pueden afectar a su rendimiento.

Sugerencias de combinación explicadas en los siguientes artículos: 
* `hint.shufflekey=<key>`y `hint.strategy=shuffle`  -  [consulta aleatoria](shufflequery.md)
* `hint.strategy=broadcast` - [combinación de difusión](broadcastjoin.md)
* `hint.remote=<strategy>` - [combinación entre clústeres](joincrosscluster.md)
