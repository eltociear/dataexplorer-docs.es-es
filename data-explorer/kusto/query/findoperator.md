---
title: buscar operador- Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe el operador find en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 4c3db23128c47c86639f15286cbcbcb748157386
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765911"
---
# <a name="find-operator"></a>Operador find

Busca filas que coinciden con un predicado en un conjunto de tablas.

::: zone pivot="azuredataexplorer"

El ámbito `find` de la también puede ser entre bases de datos o entre clústeres.

```kusto
find in (Table1, Table2, Table3) where Fruit=="apple"

find in (database('*').*) where Fruit == "apple"

find in (cluster('cluster_name').database('MyDB*'.*)) where Fruit == "apple"
```

::: zone-end

::: zone pivot="azuremonitor"

```kusto
find in (Table1, Table2, Table3) where Fruit=="apple"
```

::: zone-end

## <a name="syntax"></a>Sintaxis

* `find`[`withsource`=*ColumnName*] [`in` `(` *Table* Tabla`,` [ *Tabla*, ...] `)`] `where` *Predicado* [`project-smart`  |  `project` *ColumnName* [`:``:`*ColumnType*] [`,` *ColumnName*[*ColumnType*], ...] [`,` `pack(*)`]] 

* `find`*Predicado* `project-smart`  |  `project` [`:` *ColumnName*`,` [*ColumnType*] [ *ColumnName*[`:`*ColumnType*], ...] [`, pack(*)`]] 

## <a name="arguments"></a>Argumentos

::: zone pivot="azuredataexplorer"

* `withsource=`*ColumnName*: Opcional. De forma predeterminada, la salida incluirá una columna denominada *source_* cuyos valores indica qué tabla de origen ha contribuido cada fila. Si se especifica, *ColumnName* se usará en lugar de *source_* .
Si la consulta hace referencia a tablas de más de una base de datos (la base de datos predeterminada siempre cuenta), el valor de esta columna tendrá un nombre de tabla calificado con la base de datos. De forma similar, las cualificaciones de __clúster y base__ de datos estarán presentes en el valor si se hace referencia a más de un clúster.
* *Predicado*: [ver detalles](./findoperator.md#predicate-syntax). Expresión `boolean` [expression](./scalar-data-types/bool.md) sobre las columnas de las`,` tablas de entrada *Tabla* [ *Tabla*, ...]. Se evalúa para cada fila de cada tabla de entrada. 
* `Table`: opcional. De forma *predeterminada, find* buscará en todas las tablas de la base de datos actual
    *  El nombre de una `Events` tabla, como o
    *  Una expresión de consulta, como `(Events | where id==42)`
    *  Un conjunto de tablas especificado con un carácter comodín. Por ejemplo, `E*` formaría la unión de todas `E`las tablas de la base de datos cuyos nombres comienzan.
* `project-smart` | `project`: [ver los detalles](./findoperator.md#output-schema) si no se especifica `project-smart` se utilizará por defecto

::: zone-end

::: zone pivot="azuremonitor"

* `withsource=`*ColumnName*: Opcional. De forma predeterminada, la salida incluirá una columna denominada *source_* cuyos valores indica qué tabla de origen ha contribuido cada fila. Si se especifica, *ColumnName* se usará en lugar de *source_* .
* *Predicado*: [ver detalles](./findoperator.md#predicate-syntax). Expresión `boolean` [expression](./scalar-data-types/bool.md) sobre las columnas de las`,` tablas de entrada *Tabla* [ *Tabla*, ...]. Se evalúa para cada fila de cada tabla de entrada. 
* `Table`: opcional. De forma *predeterminada, buscar* ásearchen en todas las tablas
    *  El nombre de una `Events` tabla, como o
    *  Una expresión de consulta, como `(Events | where id==42)`
    *  Un conjunto de tablas especificado con un carácter comodín. Por ejemplo, `E*` formaría la unión de `E`todas las tablas cuyos nombres comienzan.
* `project-smart` | `project`: [ver los detalles](./findoperator.md#output-schema) si no se especifica `project-smart` se utilizará por defecto

::: zone-end

## <a name="returns"></a>Devuelve

Transformación de filas`,` en la *tabla* [ `true` *Tabla*, ...] para la que *Predicate* es . Las filas se transforman según el esquema de salida como se describe a continuación.

## <a name="output-schema"></a>Esquema de salida

**source_ columna**

La salida del operador find siempre incluirá una columna *source_* con el nombre de la tabla de origen. Se puede cambiar el `withsource` nombre de la columna mediante el parámetro.

**columnas de resultados**

Las tablas de origen que no contienen ninguna columna utilizada por la evaluación de predicados se filtrarán.

Al `project-smart`utilizar , las columnas que aparecerán en la salida serán:
1. Columnas que aparecen explícitamente en el predicado
2. Columnas que son comunes a todas las tablas filtradas El resto de las `pack_` columnas se empaquetarán en una bolsa de propiedades y aparecerán en una columna adicional.
Una columna a la que hace referencia explícitamente el predicado y aparece en varias tablas con varios tipos, tendrá una columna diferente en el esquema de resultados para cada tipo de este tipo. Cada uno de los nombres de columna se construirá a partir del nombre de columna original y el tipo, separados por un carácter de subrayado.

Cuando `project` se utiliza`:` *ColumnName*`,` [*ColumnType*] [ *ColumnName*[`:`*ColumnType*], ...] [`,` `pack(*)`]:
1. La tabla de resultados incluirá las columnas especificadas en la lista. Si una tabla de origen no contiene una columna determinada, los valores de las filas correspondientes serán nulos.
2. Al especificar un *ColumnType* junto con un *ColumnName*, esta columna en el **resultado** tendrá el tipo especificado y los valores se convertirán a ese tipo si es necesario. Tenga en cuenta que esto no tendrá ningún efecto en el tipo de columna al evaluar el *predicado*.
3. Cuando `pack(*)` se utiliza, el resto de las columnas se empaquetarán en `pack_` una bolsa de propiedades y aparecerán en una columna adicional

**pack_ columna**

Esta columna contendrá un contenedor de propiedades con los datos de todas las columnas que no aparecen en el esquema de salida. El nombre de la columna de origen servirá como el nombre de propiedad y el valor de columna servirá como el valor de propiedad.

## <a name="predicate-syntax"></a>Sintaxis de predicado

Consulte el [resumen del operador where](./whereoperator.md) para algunas funciones de filtrado.

find operator admite una `* has` sintaxis alternativa para *term* , y el uso de just *term* buscará un término en todas las columnas de entrada

## <a name="notes"></a>Notas

* Si `project` la cláusula hace referencia a una columna que aparece en varias tablas y tiene varios tipos, un tipo debe seguir esta referencia de columna en la cláusula de proyecto
* Si una columna aparece en varias `project-smart` tablas y tiene varios tipos y está `find`en uso, habrá una columna correspondiente para cada tipo en el resultado 's, como se describe en [union](./unionoperator.md)
* Cuando se utiliza *project-smart*, los cambios en el predicado, en el conjunto de tablas de origen o en el esquema de tablas pueden dar lugar a cambios en el esquema de salida. Si se necesita un esquema de resultados constante, utilice *project* en su lugar
* `find`ámbito no puede incluir [funciones](../management/functions.md). Para incluir una función en el ámbito de búsqueda - definir una [instrucción let](./letstatement.md) con [la palabra clave view](./letstatement.md)

## <a name="performance-tips"></a>Sugerencias para mejorar el rendimiento

* Utilice [tablas](../management/tables.md) en lugar de [expresiones tabulares:](./tabularexpressionstatements.md)en caso `union` de expresión tabular, el operador find vuelve a una consulta que puede dar lugar a un rendimiento degradado
* Si una columna que aparece en varias tablas y tiene varios tipos forma parte de la cláusula project, prefiere agregar un *ColumnType* a la cláusula de proyecto antes de modificar la tabla antes de pasarla a `find` (consulte la sugerencia anterior)
* Agregar filtros basados en tiempo al predicado (mediante un valor de columna datetime o [ingestion_time()](./ingestiontimefunction.md))
* Prefiere buscar en columnas específicas sobre la búsqueda de texto completo 
* Prefiere no hacer referencia explícitamente a columnas que aparecen en varias tablas y tiene varios tipos. Si el predicado es válido al resolver este tipo de columnas para más de un tipo, la consulta se recurrirá a la unión

[ver ejemplos](./findoperator.md#examples-of-cases-where-find-will-perform-as-union)

## <a name="examples"></a>Ejemplos

::: zone pivot="azuredataexplorer"

###  <a name="term-lookup-across-all-tables-in-current-database"></a>Búsqueda de términos en todas las tablas (en la base de datos actual)

La siguiente consulta busca todas las filas de todas las tablas de la base de datos actual en las que cualquier columna incluye la palabra `Kusto`. Los registros resultantes se transforman según el esquema de [salida.](#output-schema) 

```kusto
find "Kusto"
```

## <a name="term-lookup-across-all-tables-matching-a-name-pattern-in-the-current-database"></a>Búsqueda de términos en todas las tablas que coinciden con un patrón de nombre (en la base de datos actual)

La siguiente consulta busca todas las filas de todas `K`las tablas de la `Kusto`base de datos actual cuyo nombre comienza con , y en la que cualquier columna incluye la palabra .
Los registros resultantes se transforman según el esquema de [salida.](#output-schema) 

```kusto
find in (K*) where * has "Kusto"
```

###  <a name="term-lookup-across-all-tables-in-all-databases-in-the-cluster"></a>Búsqueda de términos en todas las tablas de todas las bases de datos (en el clúster)

La siguiente consulta busca todas las filas de todas las `Kusto`tablas de todas las bases de datos en las que cualquier columna incluye la palabra . Se trata de una consulta de consulta entre bases de [datos.](./cross-cluster-or-database-queries.md) Los registros resultantes se transforman según el esquema de [salida.](#output-schema) 

```kusto
find in (database('*').*) "Kusto"
```

### <a name="term-lookup-across-all-tables-and-databases-matching-a-name-pattern-in-the-cluster"></a>Búsqueda de términos en todas las tablas y bases de datos que coinciden con un patrón de nombre (en el clúster)

La siguiente consulta busca todas las filas `K` de todas las tablas cuyo nombre comienza con en todas las bases de datos cuyo nombre comienza con `B` y en las que cualquier columna incluye la palabra `Kusto`. Los registros resultantes se transforman según el esquema de [salida.](#output-schema) 

```kusto
find in (database("B*").K*) where * has "Kusto"
```

### <a name="term-lookup-in-several-clusters"></a>Búsqueda de términos en varios clústeres

La siguiente consulta busca todas las filas `K` de todas las tablas cuyo nombre comienza con en todas las bases de datos cuyo nombre comienza con `B` y en las que cualquier columna incluye la palabra `Kusto`. Los registros resultantes se transforman según el esquema de [salida.](#output-schema) 

```kusto
find in (cluster("cluster1").database("B*").K*, cluster("cluster2").database("C*".*))
where * has "Kusto"
```

::: zone-end

::: zone pivot="azuremonitor"

### <a name="term-lookup-across-all-tables"></a>Búsqueda de términos en todas las tablas

La siguiente consulta busca todas las filas de todas `Kusto`las tablas en las que cualquier columna incluye la palabra . Los registros resultantes se transforman según el esquema de [salida.](#output-schema) 

```kusto
find "Kusto"
```

::: zone-end

## <a name="examples-of-find-output-results"></a>Ejemplos `find` de resultados de resultados de producción  

En los ejemplos `find` siguientes se muestra cómo se puede usar en dos tablas: EventsTable1 y EventsTable2. Supongamos que tenemos el siguiente contenido de estas dos tablas: 

### <a name="eventstable1"></a>EventosTabla1

|Session_Id|Nivel|EventText|Versión
|---|---|---|---|
|acbd207d-51aa-4df7-bfa7-be70eb68f04e|Information|Algunos Text1|v1.0.0
|acbd207d-51aa-4df7-bfa7-be70eb68f04e|Error|Algunos Text2|v1.0.0
|28b8e46e-3c31-43cf-83cb-48921c3986fc|Error|Algunos Text3|v1.0.1
|8f057b11-3281-45c3-a856-05ebb18a3c59|Information|Algunos Text4|v1.1.0

### <a name="eventstable2"></a>EventosTabla2

|Session_Id|Nivel|EventText|EventName
|---|---|---|---|
|f7d5f95f-f580-4ea6-830b-5776c8d64fdd|Information|Otro texto1|Evento1
|acbd207d-51aa-4df7-bfa7-be70eb68f04e|Information|Algún otro texto2|Evento2
|acbd207d-51aa-4df7-bfa7-be70eb68f04e|Error|Otro texto3|Evento3
|15eaeab5-8576-4b58-8fc6-478f75d8fee4|Error|Otro texto4|Evento4


### <a name="search-in-common-columns-project-common-and-uncommon-columns-and-pack-the-rest"></a>Buscar en columnas comunes, proyectar columnas comunes y poco comunes y empaquetar el resto  

```kusto
find in (EventsTable1, EventsTable2) 
     where Session_Id == 'acbd207d-51aa-4df7-bfa7-be70eb68f04e' and Level == 'Error' 
     project EventText, Version, EventName, pack(*)
```

|source_|EventText|Versión|EventName|pack_
|---|---|---|---|---|
|EventosTabla1|Algunos Text2|v1.0.0||"Session_Id":"acbd207d-51aa-4df7-bfa7-be70eb68f04e", "Level":"Error"
|EventosTabla2|Otro texto3||Evento3|"Session_Id":"acbd207d-51aa-4df7-bfa7-be70eb68f04e", "Level":"Error"


### <a name="search-in-common-and-uncommon-columns"></a>Búsqueda en columnas comunes e infrecuentes

```kusto
find Version == 'v1.0.0' or EventName == 'Event1' project Session_Id, EventText, Version, EventName
```

|source_|Session_Id|EventText|Versión|EventName|
|---|---|---|---|---|
|EventosTabla1|acbd207d-51aa-4df7-bfa7-be70eb68f04e|Algunos Text1|v1.0.0
|EventosTabla1|acbd207d-51aa-4df7-bfa7-be70eb68f04e|Algunos Text2|v1.0.0
|EventosTabla2|f7d5f95f-f580-4ea6-830b-5776c8d64fdd|Otro texto1||Evento1

Nota: en la práctica, las filas ```Version == 'v1.0.0'``` *de EventsTable1* se filtrarán con predicado y las filas *EventsTable2* se filtrarán con ```EventName == 'Event1'``` predicado.

### <a name="use-abbreviated-notation-to-search-across-all-tables-in-the-current-database"></a>Utilice la notación abreviada para buscar en todas las tablas de la base de datos actual

```kusto
find Session_Id == 'acbd207d-51aa-4df7-bfa7-be70eb68f04e'
```

|source_|Session_Id|Nivel|EventText|pack_|
|---|---|---|---|---|
|EventosTabla1|acbd207d-51aa-4df7-bfa7-be70eb68f04e|Information|Algunos Text1|"Versión":"v1.0.0"
|EventosTabla1|acbd207d-51aa-4df7-bfa7-be70eb68f04e|Error|Algunos Text2|"Versión":"v1.0.0"
|EventosTabla2|acbd207d-51aa-4df7-bfa7-be70eb68f04e|Information|Algún otro texto2|"EventName":"Event2"
|EventosTabla2|acbd207d-51aa-4df7-bfa7-be70eb68f04e|Error|Otro texto3|"NombreDeEvento":"Evento3"


### <a name="return-the-results-from-each-row-as-a-property-bag"></a>Devolver los resultados de cada fila como una bolsa de propiedades

```kusto
find Session_Id == 'acbd207d-51aa-4df7-bfa7-be70eb68f04e' project pack(*)
```

|source_|pack_|
|---|---|
|EventosTabla1|"Session_Id":"acbd207d-51aa-4df7-bfa7-be70eb68f04e", "Level":"Information", "EventText":"Some Text1", "Version":"v1.0.0"
|EventosTabla1|"Session_Id":"acbd207d-51aa-4df7-bfa7-be70eb68f04e", "Level":"Error", "EventText":"Some Text2", "Version":"v1.0.0"
|EventosTabla2|"Session_Id":"acbd207d-51aa-4df7-bfa7-be70eb68f04e", "Level":"Information", "EventText":"Some Other Text2", "EventName":"Event2"
|EventosTabla2|"Session_Id":"acbd207d-51aa-4df7-bfa7-be70eb68f04e", "Level":"Error", "EventText":"Some Other Text3", "EventName":"Event3"


## <a name="examples-of-cases-where-find-will-perform-as-union"></a>Ejemplos de `find` casos en los que se`union`

### <a name="using-a-non-tabular-expression-as-find-operand"></a>Uso de una expresión no tabular como operando de búsqueda

```kusto
let PartialEventsTable1 = view() { EventsTable1 | where Level == 'Error' };
find in (PartialEventsTable1, EventsTable2) 
     where Session_Id == 'acbd207d-51aa-4df7-bfa7-be70eb68f04e'
```

### <a name="referencing-a-column-that-appears-in-multiple-tables-and-has-multiple-types"></a>Hacer referencia a una columna que aparece en varias tablas y tiene varios tipos

Supongamos que hemos creado dos tablas ejecutando: 

```kusto
.create tables 
  Table1 (Level:string, Timestamp:datetime, ProcessId:string),
  Table2 (Level:string, Timestamp:datetime, ProcessId:int64)
```

* La siguiente consulta se `union`ejecutará como:
```kusto
find in (Table1, Table2) where ProcessId == 1001
```

Y el esquema de resultado de salida será __(Level:string, Timestamp, ProcessId_string, ProcessId_int)__

* La siguiente consulta también se ejecutará `union` como pero producirá un esquema de resultados diferente:
```kusto
find in (Table1, Table2) where ProcessId == 1001 project Level, Timestamp, ProcessId:string 
```

Y el esquema de resultado de salida será __(Level:string, Timestamp, ProcessId_string)__
