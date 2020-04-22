---
title: operador de unión- Azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe el operador de unión en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: b62c259e1abf1ff0e0e98a90ac4da5a000db5320
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765409"
---
# <a name="union-operator"></a>Operador union

Toma dos o más tablas y devuelve las filas de todas ellas. 

```kusto
Table1 | union Table2, Table3
```

**Sintaxis**

*T* `| union` [*UnionParameters*`kind=` `inner` | `outer`]`withsource=`[ ] [`,` *ColumnName*`isfuzzy=` `true` | `false`] [ ] *Tabla* [ *Tabla*]...  

Forma alternativa sin entrada en tubáca:

`union`[*UnionParameters*] [`kind=` `inner``isfuzzy=` `true` | `false` *Table* `,` *Table**ColumnName*`withsource=`] [ ColumnName ] [ ] Tabla [ Tabla ]... | `outer`  

**Argumentos**

::: zone pivot="azuredataexplorer"

* `Table`:
    *  El nombre de una tabla, como `Events`; o
    *  Expresión de consulta que debe incluirse entre paréntesis, como `(Events | where id==42)` o `(cluster("https://help.kusto.windows.net:443").database("Samples").table("*"))`; O
    *  Un conjunto de tablas especificado con un carácter comodín. Por ejemplo, `E*` formaría la unión de todas `E`las tablas de la base de datos cuyos nombres comienzan.
* `kind`: 
    * `inner` : el resultado tiene el subconjunto de columnas que son comunes a todas las tablas de entrada.
    * `outer`- (predeterminado). El resultado tiene todas las columnas que se producen en cualquiera de las entradas. Las celdas que no se definieron `null`mediante una fila de entrada se establecen en .
* `withsource`=*ColumnName*: Si se especifica, la salida incluirá una columna denominada *ColumnName* cuyo valor indica qué tabla de origen ha contribuido con cada fila.
Si la consulta hace referencia a tablas de más de una base de datos (la base de datos predeterminada siempre cuenta), el valor de esta columna tendrá un nombre de tabla calificado con la base de datos.
De forma similar, las cualificaciones de __clúster y base__ de datos estarán presentes en el valor si se hace referencia a más de un clúster. 
* `isfuzzy=``true` `isfuzzy` : Si se `true` establece en - permite la resolución difusa de las patas de unión.  |  `false` `Fuzzy`se aplica al `union` conjunto de fuentes. Esto significa que al analizar la consulta y prepararse para la ejecución, el conjunto de orígenes de unión se reduce al conjunto de referencias de tabla que existen y son accesibles en ese momento. Si se encontró al menos una de estas tablas, cualquier error de resolución producirá una advertencia en los resultados de estado de la consulta (una para cada referencia que falta), pero no impedirá la ejecución de la consulta; si no hay resoluciones correctas, la consulta devolverá un error.
De manera predeterminada, es `isfuzzy=` `false`.
* *UnionParameters*: Cero o más parámetros (separados por espacios) en forma de *valor* de *nombre* `=` que controlan el comportamiento de la operación de coincidencia de filas y el plan de ejecución. Se admiten los siguientes parámetros: 

  |Nombre           |Valores                                        |Descripción                                  |
  |---------------|----------------------------------------------|---------------------------------------------|
  |`hint.concurrency`|*Número*|Sugerencias al sistema cuántas subconsultas simultáneas del `union` operador se deben ejecutar en paralelo. *Predeterminado:* Cantidad de núcleos de CPU en el nodo único del clúster (2 a 16).|
  |`hint.spread`|*Number*|Sugerencias al sistema cuántos nodos debe `union` utilizar la ejecución simultánea de subconsultas. *Predeterminado*: 1.|

::: zone-end

::: zone pivot="azuremonitor"

* `Table`:
    *  El nombre de una tabla, como`Events`
    *  Una expresión de consulta que debe incluirse entre paréntesis, como`(Events | where id==42)`
    *  Un conjunto de tablas especificado con un carácter comodín. Por ejemplo, `E*` formaría la unión de todas `E`las tablas de la base de datos cuyos nombres comienzan.
* `kind`: 
    * `inner` : el resultado tiene el subconjunto de columnas que son comunes a todas las tablas de entrada.
    * `outer`- (predeterminado). El resultado tiene todas las columnas que se producen en cualquiera de las entradas. Las celdas que no se definieron `null`mediante una fila de entrada se establecen en .
* `withsource`=*ColumnName*: Si se especifica, la salida incluirá una columna denominada *ColumnName* cuyo valor indica qué tabla de origen aportó cada fila.
Si la consulta hace referencia a tablas de más de una base de datos (la base de datos predeterminada siempre cuenta), el valor de esta columna tendrá un nombre de tabla calificado con la base de datos.
De forma similar, las cualificaciones de __clúster y base__ de datos estarán presentes en el valor si se hace referencia a más de un clúster. 
* `isfuzzy=``true` `isfuzzy` : Si se `true` establece en - permite la resolución difusa de las patas de unión.  |  `false` `Fuzzy`se aplica al `union` conjunto de fuentes. Esto significa que al analizar la consulta y prepararse para la ejecución, el conjunto de orígenes de unión se reduce al conjunto de referencias de tabla que existen y son accesibles en ese momento. Si se encontró al menos una de estas tablas, cualquier error de resolución producirá una advertencia en los resultados de estado de la consulta (una para cada referencia que falta), pero no impedirá la ejecución de la consulta; si no hay resoluciones correctas, la consulta devolverá un error.
De manera predeterminada, es `isfuzzy=false`.

::: zone-end

**Devuelve**

Una tabla con todas las filas que hay en las tablas de entrada.

**Notas**

::: zone pivot="azuredataexplorer"

1. `union`ámbito puede incluir [instrucciones let](./letstatement.md) si se atribuyen con la palabra [clave view](./letstatement.md)
2. `union`el ámbito no incluirá [funciones.](../management/functions.md) Para incluir una función en el ámbito de unión, defina una [instrucción let](./letstatement.md) con la [palabra clave view](./letstatement.md)
3. Si `union` la entrada es [tablas](../management/tables.md) (a `union` diferencia de las [expresiones tabulares)](./tabularexpressionstatements.md)y el es seguido por un operador [where](./whereoperator.md), para un mejor rendimiento, considere la posibilidad de reemplazar ambos con [find](./findoperator.md). Tenga en cuenta el esquema `find` de [salida](./findoperator.md#output-schema) diferente generado por el operador. 
4. `isfuzzy=true`sólo se `union` aplica a la fase de resolución de fuentes. Una vez determinado el conjunto de tablas de origen, no se suprimirán los posibles errores de consulta adicionales.
5. Cuando `outer union`se utiliza , el resultado tiene todas las columnas que se producen en cualquiera de las entradas, una columna para cada nombre y tipo de apariciones. Esto significa que si una columna aparece en varias tablas y tiene varios `union`tipos, tendrá una columna correspondiente para cada tipo en el resultado 's. Este nombre de columna se supondrá con un '_' seguido del [tipo](./scalar-data-types/index.md)de columna de origen .

::: zone-end

::: zone pivot="azuremonitor"

1. `union`ámbito puede incluir [instrucciones let](./letstatement.md) si se atribuyen con la palabra [clave view](./letstatement.md)
2. `union`alcance no incluirá funciones. Para incluir la función en el ámbito de unión - definir una [instrucción let](./letstatement.md) con [la palabra clave view](./letstatement.md)
3. Si `union` la entrada es tablas (a `union` diferencia de las [expresiones tabulares)](./tabularexpressionstatements.md)y el es seguido por un operador [where](./whereoperator.md), considere la posibilidad de reemplazar ambos con [find](./findoperator.md) para un mejor rendimiento. Tenga en cuenta los diferentes `find` esquemas de [salida](./findoperator.md#output-schema) producidos por el operador. 
4. `isfuzzy=``true` sólo se aplica a `union` la fase de la resolución de fuentes. Una vez determinado el conjunto de tablas de origen, no se suprimirán los posibles errores de consulta adicionales.
5. Cuando `outer union`se utiliza , el resultado tiene todas las columnas que se producen en cualquiera de las entradas, una columna para cada nombre y tipo de apariciones. Esto significa que si una columna aparece en varias tablas y tiene varios `union`tipos, tendrá una columna correspondiente para cada tipo en el resultado 's. Este nombre de columna se supondrá con un '_' seguido del [tipo](./scalar-data-types/index.md)de columna de origen .

::: zone-end


**Ejemplo**

```kusto
union K* | where * has "Kusto"
```

Filas de todas las tablas `K`de la base de datos `Kusto`cuyo nombre comienza con , y en las que cualquier columna incluye la palabra .

**Ejemplo**

```kusto
union withsource=SourceTable kind=outer Query, Command
| where Timestamp > ago(1d)
| summarize dcount(UserId)
```

El número de usuarios distintivos que han creado un evento `Query` o un evento `Command` a lo largo del día anterior. En el resultado, la columna "SourceTable" indicará "Query" o "Command".

```kusto
Query
| where Timestamp > ago(1d)
| union withsource=SourceTable kind=outer 
   (Command | where Timestamp > ago(1d))
| summarize dcount(UserId)
```

Esta versión más eficaz, produce el mismo resultado. Filtra cada tabla antes de crear la unión.

**Ejemplo: Uso de`isfuzzy=true`**
 
```kusto     
// Using union isfuzzy=true to access non-existing view:                                     
let View_1 = view () { print x=1 };
let View_2 = view () { print x=1 };
let OtherView_1 = view () { print x=1 };
union isfuzzy=true
(View_1 | where x > 0), 
(View_2 | where x > 0),
(View_3 | where x > 0)
| count 
```

|Count|
|---|
|2|

Observación del estado de la consulta: se devuelve la siguiente advertencia:`Failed to resolve entity 'View_3'`

```kusto
// Using union isfuzzy=true and wildcard access:
let View_1 = view () { print x=1 };
let View_2 = view () { print x=1 };
let OtherView_1 = view () { print x=1 };
union isfuzzy=true View*, SomeView*, OtherView*
| count 
```

|Count|
|---|
|3|

Observación del estado de la consulta: se devuelve la siguiente advertencia:`Failed to resolve entity 'SomeView*'`

**Ejemplo: los tipos de columnas de origen no coinciden**
 
```kusto     
let View_1 = view () { print x=1 };
let View_2 = view () { print x=toint(2) };
union withsource=TableName View_1, View_2
```

|TableName|x_long|x_int|
|---------|------|-----|
|View_1   |1     |     |
|View_2   |      |2    |

```kusto     
let View_1 = view () { print x=1 };
let View_2 = view () { print x=toint(2) };
let View_3 = view () { print x_long=3 };
union withsource=TableName View_1, View_2, View_3 
```

|TableName|x_long1|x_int |x_long|
|---------|-------|------|------|
|View_1   |1      |      |      |
|View_2   |       |2     |      |
|View_3   |       |      |3     |

Columna `x` `View_1` de recibido `_long`el sufijo , `x_long` y como una columna nombrada ya existe en el esquema de resultados, los nombres de columna se desduplicaron, produciendo una nueva columna-`x_long1`
