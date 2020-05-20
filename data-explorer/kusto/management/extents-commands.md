---
title: 'Administración de extensiones (particiones de datos): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la administración de extensiones (particiones de datos) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: ca66beaad41796dff38a5bd5ce1fad2e0f41fc72
ms.sourcegitcommit: 974d5f2bccabe504583e387904851275567832e7
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/18/2020
ms.locfileid: "83550510"
---
# <a name="extents-data-shards-management"></a>Administración de extensiones (particiones de datos)

Las particiones de datos se denominan **extensiones** en Kusto y todos los comandos usan "extensión" o "extensiones" como sinónimo.

## <a name="show-extents"></a>. Mostrar extensiones

**Nivel de clúster**

`.show` `cluster` `extents` [`hot`]

Muestra información sobre las extensiones (particiones de datos) que se encuentran en el clúster.
Si `hot` se especifica, muestra solo las extensiones que se espera que estén en la caché activa.

**Nivel de base de datos**

`.show``database` *DatabaseName* `extents` [ `(` *ExtentId1* `,` ... `,` *ExtentIdN* `)` ] [ `hot` ] [ `where` `tags` ( `has` | `contains` | `!has` | `!contains` ) *Tag1* [ `and` `tags` ( `has` | `contains` | `!has` | `!contains` ) *etiqueta2*...]]

Muestra información sobre las extensiones (particiones de datos) que se encuentran en la base de datos especificada.
Si `hot` se especifica, muestra solo las extensiones que se espera que estén en la caché activa.

**Nivel de tabla**

`.show``table` *TableName* `extents` [ `(` *ExtentId1* `,` ... `,` *ExtentIdN* `)` ] [ `hot` ] [ `where` `tags` ( `has` | `contains` | `!has` | `!contains` ) *Tag1* [ `and` `tags` ( `has` | `contains` | `!has` | `!contains` ) *etiqueta2*...]]

`.show``tables` `(` *TableName1* `,` ... `,` *TableNameN* `)` `extents`[ `(` *ExtentId1* `,` ... `,` *ExtentIdN* `)` ] [ `hot` ] [ `where` `tags` ( `has` | `contains` | `!has` | `!contains` ) *Tag1* [ `and` `tags` ( `has` | `contains` | `!has` | `!contains` ) *etiqueta2*...]]

Muestra información sobre las extensiones (particiones de datos) que se encuentran en las tablas especificadas (la base de datos se toma del contexto del comando).
Si `hot` se especifica, muestra solo las extensiones que se espera que estén en la caché activa.

**NOTAS**

* El uso de comandos de nivel de base de datos o de tabla es mucho más rápido que el filtrado (adición `| where DatabaseName == '...' and TableName == '...'` ) de los resultados de un comando de nivel de clúster.
* Si se proporciona la lista opcional de identificadores de extensión, el conjunto de datos devueltos se limita solo a esas extensiones.
    * De nuevo, esto es mucho más rápido que filtrar (agregar `| where ExtentId in(...)` a) los resultados de los comandos "sin sistema operativo".
* En caso de que `tags` se especifiquen filtros:
  * La lista devuelta se limita a aquellas extensiones cuya colección de etiquetas obedece a *todos* los filtros de etiquetas proporcionados.
    * De nuevo, esto es mucho más rápido que filtrar (agregar `| where Tags has '...' and Tags contains '...'` ) los resultados de los comandos "sin sistema operativo".
  * `has`los filtros son filtros de igualdad: se filtrarán las extensiones que no estén etiquetadas con ninguna de las etiquetas especificadas.
  * `!has`los filtros son filtros negativos de igualdad: las extensiones que se etiquetan con cualquiera de las etiquetas especificadas se filtrarán.
  * `contains`los filtros son filtros de subcadena sin distinción entre mayúsculas y minúsculas: las extensiones que no tienen las cadenas especificadas como una subcadena de cualquiera de sus etiquetas se filtrarán. 
  * `!contains`los filtros distinguen los filtros de subcadenas negativos de mayúsculas y minúsculas: las extensiones que tienen las cadenas especificadas como una subcadena de cualquiera de sus etiquetas se filtrarán. 
  
  * **Ejemplos**
    * La extensión `E` de la tabla `T` se etiqueta con etiquetas `aaa` , `BBB` y `ccc` .
    * Esta consulta devolverá `E` : 
    
    ```kusto 
    .show table T extents where tags has 'aaa' and tags contains 'bb' 
    ``` 
    
    * Esta consulta *no* devolverá `E` (ya que no se etiqueta con una etiqueta que sea igual a `aa` ):
    
    ```kusto 
    .show table T extents where tags has 'aa' and tags contains 'bb' 
    ``` 
    
    * Esta consulta devolverá `E` : 
    
    ```kusto 
    .show table T extents where tags contains 'aaa' and tags contains 'bb' 
    ``` 

|Parámetro de salida |Tipo |Descripción 
|---|---|---
|ExtentId |Guid |IDENTIFICADOR de la extensión 
|DatabaseName |String |Base de datos a la que pertenece la extensión 
|TableName |String |Tabla a la que pertenecen las extensiones 
|MaxCreatedOn |DateTime |Fecha y hora en que se creó la extensión (para una extensión combinada: el máximo de tiempos de creación entre las extensiones de origen) 
|OriginalSize |Doble |Tamaño original en bytes de los datos de extensión 
|ExtentSize |Doble |Tamaño de la extensión en memoria (comprimido + índice) 
|CompressedSize |Doble |Tamaño comprimido de los datos de extensión en memoria 
|IndexSize |Doble |Tamaño de índice de los datos de extensión 
|Bloques |long |Cantidad de bloques de datos en extensión 
|Segmentos |long |Cantidad de segmento de datos en extensión 
|AssignedDataNodes |String | En desuso (una cadena vacía)
|LoadedDataNodes |String |En desuso (una cadena vacía)
|ExtentContainerId |String | IDENTIFICADOR del contenedor de extensión en el que se encuentra la extensión
|RowCount |long |Cantidad de filas en la extensión
|MinCreatedOn |DateTime |Fecha y hora en que se creó la extensión (para una extensión combinada: el tiempo mínimo de creación entre las extensiones de origen) 
|Etiquetas|String|Etiquetas, si las hay, definidas para la extensión 
 
**Ejemplos**

```kusto 
// Show volume of extents being created per hour in a specific database
.show database MyDatabase extents | summarize count(ExtentId) by MaxCreatedOn bin=time(1h) | render timechart  

// Show volume of data arriving by table per hour 
.show database MyDatabase extents  
| summarize sum(OriginalSize) by TableName, MaxCreatedOn bin=time(1h)  
| render timechart

// Show data size distribution by table 
.show database MyDatabase extents | summarize sum(OriginalSize) by TableName

// Show all extents in the database named 'GamesDB' 
.show database GamesDB extents

// Show all extents in the table named 'Games' 
.show table Games extents

// Show all extents in the tables named 'TaggingGames1' and 'TaggingGames2', tagged with both 'tag1' and 'tag2' 
.show tables (TaggingGames1,TaggingGames2) extents where tags has 'tag1' and tags has 'tag2'
``` 
 
## <a name="merge-extents"></a>. extensiones de combinación

**Sintaxis**

`.merge``[async | dryrun]` *TableName* `(` *GUID1* [ `,` *GUID2* ...] `)``[with(rebuild=true)]`

Este comando combina las extensiones (consulte [información general sobre las extensiones de datos](extents-overview.md)) indicadas por sus identificadores en la tabla especificada.

Hay dos tipos de operaciones de combinación: *Merge* (que vuelve a generar los índices) y *rebuild (que renueva* los datos por completo).

* `async`: La operación se realizará de forma asincrónica. el resultado será un identificador de operación (GUID), que se puede ejecutar `.show operations <operation ID>` con, para ver el estado del comando & estado.
* `dryrun`: La operación solo mostrará las extensiones que se deben combinar, pero realmente no las combinará. 
* `with(rebuild=true)`: se volverán a generar las extensiones (los datos se volverán a introducir) en lugar de combinarse (los índices se volverán a generar).

**Devolver salida**

Parámetro de salida |Tipo |Descripción 
---|---|---
OriginalExtentId |string |Identificador único (GUID) para la extensión original en la tabla de origen, que se ha combinado en la extensión de destino. 
ResultExtentId |string |Identificador único (GUID) para la extensión del resultado que se ha creado a partir de las extensiones de origen. En caso de error: "Failed" o "Abandoned".
Duration |timespan |El período de tiempo que se tardó en completar la operación de combinación.

**Ejemplos**

Vuelve a generar dos extensiones específicas en, que se `MyTable` realizan de forma asincrónica.
```kusto
.merge async MyTable (e133f050-a1e2-4dad-8552-1f5cf47cab69, 0d96ab2d-9dd2-4d2c-a45e-b24c65aa6687) with(rebuild=true)
```

Combina dos extensiones específicas en `MyTable` , realizadas sincrónicamente
```kusto
.merge MyTable (12345050-a1e2-4dad-8552-1f5cf47cab69, 98765b2d-9dd2-4d2c-a45e-b24c65aa6687)
```

**Notas**
- En general, `.merge` los comandos se deben ejecutar de forma manual y se ejecutan continuamente en segundo plano en el clúster de Kusto (de acuerdo con las [directivas de combinación](mergepolicy.md) definidas para las tablas y bases de datos que contengan).  
  - Las consideraciones generales sobre los criterios para combinar varias extensiones en una sola se documentan en [combinación de directivas](mergepolicy.md).
- `.merge`las operaciones tienen un posible estado final de `Abandoned` (que se puede ver cuando se ejecuta `.show operations` con el identificador de operación). este estado sugiere que las extensiones de origen no se combinaron juntas, ya que algunas de las extensiones de origen ya no existen en la tabla de origen.
  - Se espera que este estado se produzca en casos como:
     - Las extensiones de origen se han quitado como parte de la retención de la tabla.
     - Las extensiones de origen se han pasado a una tabla diferente.
     - La tabla de origen se ha quitado o cambiado de nombre completamente.

## <a name="move-extents"></a>. migrar extensiones

**Sintaxis**

`.move`[ `async` ] `extents` `all` `from` `table` *SourceTableName* `to` `table` *DestinationTableName*

`.move`[ `async` ] `extents` `(` *GUID1* [ `,` *GUID2* ...] `)` `from` `table` *SourceTableName* `to` `table` *DestinationTableName* 

`.move`[ `async` ] `extents` `to` `table` *DestinationTableName*  <|  *consulta*

Este comando se ejecuta en el contexto de una base de datos específica y mueve las extensiones especificadas de la tabla de origen a la tabla de destino de transacciones.
Requiere el [permiso de administrador de tablas](../management/access-control/role-based-authorization.md) para las tablas de origen y de destino.

* `async`(opcional) especifica si el comando se ejecuta de forma asincrónica (en cuyo caso, se devuelve un identificador de operación (GUID) y el estado de la operación se puede supervisar mediante el comando [. show Operations](operations.md#show-operations) ).
    * En caso de que se use esta opción, se puede recuperar el resultado de la ejecución correcta del comando [. Mostrar detalles](operations.md#show-operation-details) de la operación.

Hay tres maneras de especificar qué extensiones se van a trasladar:
1. Se mueven todas las extensiones de una tabla específica.
2. Especificando explícitamente los identificadores de extensión en la tabla de origen.
3. Al proporcionar una consulta cuyos resultados especifican los identificadores de extensión en las tablas de origen.

**Restricciones**
- Las tablas de origen y de destino deben estar en la base de datos de contexto. 
- Se espera que todas las columnas de la tabla de origen existan en la tabla de destino con el mismo nombre y tipo de datos.

**Especificar extensiones con una consulta**

```kusto 
.move extents to table TableName <| ...query... 
```

Las extensiones se especifican mediante una consulta Kusto que devuelve un conjunto de registros con una columna denominada "ExtentId". 

**Devolver salida** (para la ejecución de sincronización)

Parámetro de salida |Tipo |Descripción 
---|---|---
OriginalExtentId |string |Un identificador único (GUID) para la extensión original en la tabla de origen, que se ha pasado a la tabla de destino. 
ResultExtentId |string |Identificador único (GUID) para la extensión del resultado que se ha pasado de la tabla de origen a la tabla de destino. En caso de error: "Failed".
Detalles |string |Incluye los detalles del error, en caso de que se produzca un error en la operación.

**Ejemplos**

Mueve todas las extensiones de la tabla `MyTable` a la tabla `MyOtherTable` .
```kusto
.move extents all from table MyTable to table MyOtherTable
```

Mueve 2 extensiones específicas (por sus identificadores de extensión) de la tabla `MyTable` a la tabla `MyOtherTable` .
```kusto
.move extents (AE6CD250-BE62-4978-90F2-5CB7A10D16D7,399F9254-4751-49E3-8192-C1CA78020706) from table MyTable to table MyOtherTable
```

Mueve todas las extensiones de 2 tablas específicas ( `MyTable1` , `MyTable2` ) a la tabla `MyOtherTable` .
```kusto
.move extents to table MyOtherTable <| .show tables (MyTable1,MyTable2) extents
```

**Salida del ejemplo** 

|OriginalExtentId |ResultExtentId| Detalles
|---|---|---
|e133f050-a1e2-4dad-8552-1f5cf47cab69 |0d96ab2d-9dd2-4d2c-a45e-b24c65aa6687| 
|cdbeb35b-87ea-499f-b545-defbae091b57 |a90a303c-8a14-4207-8f35-d8ea94ca45be| 
|4fcb4598-9a31-4614-903c-0c67c286da8c |97aafea1-59ff-4312-b06b-08f42187872f| 
|2dfdef64-62a3-4950-a130-96b5b1083b5a |0fb7f3da-5e28-4f09-a000-e62eb41592df| 

## <a name="drop-extents"></a>. Drop (extensiones)

Quita extensiones de la base de datos o tabla especificada. Este comando tiene varias variantes: en una variante las extensiones que se van a quitar se especifican mediante una consulta Kusto, mientras que en las otras extensiones de variantes se especifican mediante un minilenguaje que se describe a continuación. 
 
### <a name="specifying-extents-with-a-query"></a>Especificar extensiones con una consulta

Requiere el [permiso de administrador de tabla](../management/access-control/role-based-authorization.md) foreach de las tablas que tienen extensiones devueltas por la consulta proporcionada.

Quita las extensiones (o simplemente informa de ellas sin quitar realmente si `whatif` se usa):

**Sintaxis**

`.drop``extents`[ `whatif` ] <| *consulta* de

Las extensiones se especifican mediante una consulta Kusto que devuelve un conjunto de registros con una columna denominada "ExtentId". 
 
### <a name="dropping-a-specific-extent"></a>Quitar una extensión específica

Requiere el [permiso administrador de tablas](../management/access-control/role-based-authorization.md) en caso de que se especifique el nombre de la tabla.

Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md) en caso de que no se especifique el nombre de tabla.

**Sintaxis**

`.drop``extent` *ExtentId* [ `from` *TableName*]

### <a name="dropping-specific-multiple-extents"></a>Quitar varias extensiones específicas

Requiere el [permiso administrador de tablas](../management/access-control/role-based-authorization.md) en caso de que se especifique el nombre de la tabla.

Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md) en caso de que no se especifique el nombre de tabla.

**Sintaxis**

`.drop``extents` `(` *ExtentId1* `,` ... *ExtentIdN* `)` [ `from` *TableName*]

### <a name="specifying-extents-by-properties"></a>Especificar extensiones por propiedades

Requiere el [permiso administrador de tablas](../management/access-control/role-based-authorization.md) en caso de que se especifique el nombre de la tabla.

Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md) en caso de que no se especifique el nombre de tabla.

`.drop``extents`[ `older` *N* ( `days`  |  `hours` )] `from` (*TableName*  |  `all` `tables` ) [ `trim` `by` ( `extentsize`  |  `datasize` ) *N* ( `MB`  |  `GB`  |  `bytes` )] [ `limit` *LimitCount*]

* `older`: Solo se quitarán las extensiones anteriores a *N* días/horas. 
* `trim`: La operación recortará los datos de la base de datos hasta que la suma de las extensiones coincida con el tamaño deseado (MaxSize). 
* `limit`: La operación se aplicará a las primeras extensiones de *LimitCount* 

El comando admite el modo de emulación ( `.drop-pretend` en lugar de `.drop` ), que produce una salida como si se hubiera ejecutado el comando, pero sin ejecutarlo realmente.

**Ejemplos**

Quitar todas las extensiones creadas hace más de 10 días desde todas las tablas de la base de datos`MyDatabase`

```kusto
.drop extents <| .show database MyDatabase extents | where CreatedOn < now() - time(10d)
```

Quita todas las extensiones de las tablas `Table1` y `Table2` , cuya hora de creación fue de hace más de 10 días:

```kusto
.drop extents older 10 days from tables (Table1, Table2)
```

Modo de emulación: muestra las extensiones que el comando quitará (el parámetro de identificador de extensión no es aplicable a este comando):

```kusto
.drop-pretend extents older 10 days from all tables
```

Quita todas las extensiones de ' Tablaprueba ':

```kusto
.drop extents from TestTable
```
 
**Devolver salida**

|Parámetro de salida |Tipo |Descripción 
|---|---|---
|ExtentId |String |ExtentId que se quitó como resultado del comando 
|TableName |String |Nombre de tabla, donde pertenecía la extensión  
|CreatedOn |DateTime |Marca de tiempo que contiene información sobre cuándo se creó inicialmente la extensión 
 
**Salida del ejemplo** 

|Identificador de extensión |Nombre de la tabla |Creada el 
|---|---|---
|43c6e03f-1713-4ca7-a52a-5db8a4e8b87d |TestTable |2015-01-12 12:48:49.4298178 

## <a name="replace-extents"></a>. reemplazar extensiones

**Sintaxis**

`.replace`[ `async` ] `extents` `in` `table` *DestinationTableName* `<| 
{` *consultar las extensiones que se van a quitar de*la `},{` *consulta de tabla para las extensiones que se van a pasar a la tabla*`}`

Este comando se ejecuta en el contexto de una base de datos concreta, mueve las extensiones especificadas de las tablas de origen a la tabla de destino y quita las extensiones especificadas de la tabla de destino.
Todas las operaciones de eliminación y movimiento se realizan en una sola transacción.

Requiere el [permiso de administrador de tablas](../management/access-control/role-based-authorization.md) para las tablas de origen y de destino.

* `async`(opcional) especifica si el comando se ejecuta de forma asincrónica (en cuyo caso, se devuelve un identificador de operación (GUID) y el estado de la operación se puede supervisar mediante el comando [. show Operations](operations.md#show-operations) ).
    * En caso de que se use esta opción, se puede recuperar el resultado de la ejecución correcta del comando [. Mostrar detalles](operations.md#show-operation-details) de la operación.

La especificación de las extensiones que se deben quitar o descartar se realiza proporcionando 2 consultas
- *consultar las extensiones que se van a quitar de la tabla* : los resultados de esta consulta especifican los identificadores de extensión  
que se debe quitar de la tabla de destino.
- *consultar las extensiones que se van a pasar a la tabla* : los resultados de esta consulta especifican los identificadores de extensión de las tablas de origen que deben moverse a la tabla de destino.

Ambas consultas deben devolver un conjunto de registros con una columna denominada "ExtentId".

**Restricciones**
- Las tablas de origen y de destino deben estar en la base de datos de contexto. 
- Se espera que todas las extensiones especificadas por la *consulta para las extensiones que se van a quitar de la tabla* pertenezcan a la tabla de destino.
- Se espera que todas las columnas de las tablas de origen existan en la tabla de destino con el mismo nombre y tipo de datos.

**Devolver salida** (para la ejecución de sincronización)

Parámetro de salida |Tipo |Descripción 
---|---|---
OriginalExtentId |string |Identificador único (GUID) para la extensión original en la tabla de origen, que se ha descartado a la tabla de destino, o-la extensión de la tabla de destino que se ha quitado.
ResultExtentId |string |Identificador único (GUID) de la extensión del resultado que se ha descartado de la tabla de origen a la tabla de destino, o bien, vacío, en caso de que la extensión se haya quitado de la tabla de destino. En caso de error: "Failed".
Detalles |string |Incluye los detalles del error, en caso de que se produzca un error en la operación.

> [!NOTE]
> El comando generará un error si las extensiones devueltas por las *extensiones que se van a quitar de* la consulta de tabla no existen en la tabla de destino. Esto puede ocurrir si las extensiones se combinaron antes de que se ejecutara el comando Replace. Para asegurarse de que el comando produce un error en las extensiones que faltan, compruebe que la consulta devuelve el ExtentIds esperado. En #1 de ejemplo siguiente se producirá un error si la extensión que se va a quitar no existe en la tabla MyOtherTable. Sin embargo, #2 de ejemplo se realizarán correctamente aunque no exista la extensión que se va a quitar, ya que la consulta que se va a quitar no devolvió ningún ID. de extensión. 

**Ejemplos**

El siguiente comando mueve todas las extensiones de 2 tablas específicas ( `MyTable1` , `MyTable2` ) a la tabla `MyOtherTable` y quita todas las extensiones en `MyOtherTable` etiquetadas con `drop-by:MyTag` :

```kusto
.replace extents in table MyOtherTable <|
    {
        .show table MyOtherTable extents where tags has 'drop-by:MyTag'
    },
    {
        .show tables (MyTable1,MyTable2) extents
    }
```

**Salida del ejemplo** 

|OriginalExtentId |ResultExtentId |Detalles
|---|---|---
|e133f050-a1e2-4dad-8552-1f5cf47cab69 |0d96ab2d-9dd2-4d2c-a45e-b24c65aa6687| 
|cdbeb35b-87ea-499f-b545-defbae091b57 |a90a303c-8a14-4207-8f35-d8ea94ca45be| 
|4fcb4598-9a31-4614-903c-0c67c286da8c |97aafea1-59ff-4312-b06b-08f42187872f| 
|2dfdef64-62a3-4950-a130-96b5b1083b5a |0fb7f3da-5e28-4f09-a000-e62eb41592df| 


Los siguientes comandos mueven todas las extensiones de una tabla específica ( `MyTable1` ) a `MyOtherTable` la tabla y quita una extensión específica en `MyOtherTable` , por su identificador:


```kusto
.replace extents in table MyOtherTable <|
    {
        print ExtentId = "2cca5844-8f0d-454e-bdad-299e978be5df"
    },
    {
        .show table MyTable1 extents 
    }
```

```kusto
.replace extents in table MyOtherTable  <|
    {
        .show table MyOtherTable extents
        | where ExtentId == guid(2cca5844-8f0d-454e-bdad-299e978be5df) 
    },
    {
        .show table MyTable1 extents 
    }
```

El siguiente comando implementa una lógica idempotente, de modo que solo quita las extensiones de la tabla `t_dest` en caso de que haya extensiones para moverse de una tabla `t_source` a otra `t_dest` :

```kusto
.replace async extents in table t_dest <|
{
    let any_extents_to_move = toscalar( 
        t_source
        | where extent_tags() has 'drop-by:blue'
        | summarize count() > 0
    );
    let extents_to_drop =
        t_dest
        | where any_extents_to_move and extent_tags() has 'drop-by:blue'
        | summarize by ExtentId = extent_id()
    ;
    extents_to_drop
},
{
    let extents_to_move = 
        t_source
        | where extent_tags() has 'drop-by:blue'
        | summarize by ExtentId = extent_id()
    ;
    extents_to_move
}
```

## <a name="drop-extent-tags"></a>. Drop extent (etiquetas)

**Sintaxis**

`.drop`[ `async` ] `extent` `tags` `from` `table` *TableName* `(` '*Tag1*' [ `,` '*etiqueta2*' `,` ... `,` ' *TagN*']`)`

`.drop`[ `async` ] `extent` `tags`  <|  *consulta*

* `async`(opcional) especifica si el comando se ejecuta de forma asincrónica (en cuyo caso, se devuelve un identificador de operación (GUID) y el estado de la operación se puede supervisar mediante el comando [. show Operations](operations.md#show-operations) ).
    * En caso de que se use esta opción, se puede recuperar el resultado de la ejecución correcta del comando [. Mostrar detalles](operations.md#show-operation-details) de la operación.

El comando se ejecuta en el contexto de una base de datos específica y quita las [etiquetas de extensión](extents-overview.md#extent-tagging) proporcionadas de cualquier extensión en la base de datos y la tabla proporcionadas, que incluye cualquiera de las etiquetas.  

Hay dos maneras de especificar las etiquetas que deben quitarse de las extensiones:

1. Especificando explícitamente las etiquetas que deben quitarse de todas las extensiones de la tabla especificada.
2. Al proporcionar una consulta cuyos resultados especifican los identificadores de extensión en la tabla y la extensión foreach, las etiquetas que se deben quitar.

**Restricciones**
- Todas las extensiones deben estar en la base de datos de contexto y deben pertenecer a la misma tabla. 

**Especificar extensiones con una consulta**

Requiere el [permiso de administrador de tablas](../management/access-control/role-based-authorization.md) para todas las tablas de origen y de destino implicadas.

```kusto 
.drop extent tags <| ...query... 
```

Las extensiones y las etiquetas que se van a quitar se especifican mediante una consulta Kusto que devuelve un conjunto de registros con una columna denominada "ExtentId" y una columna denominada "Tags". 

*Nota*: cuando se usa la [biblioteca de cliente de Kusto .net](../api/netfx/about-kusto-data.md), puede usar los métodos siguientes para generar el comando deseado:
- `CslCommandGenerator.GenerateExtentTagsDropByRegexCommand(string tableName, string regex)`
- `CslCommandGenerator.GenerateExtentTagsDropBySubstringCommand(string tableName, string substring)`

**Devolver salida**

Parámetro de salida |Tipo |Descripción 
---|---|---
OriginalExtentId |string |Identificador único (GUID) de la extensión original cuyas etiquetas se han modificado (y se han quitado como parte de la operación). 
ResultExtentId |string |Identificador único (GUID) para la extensión del resultado que ha modificado las etiquetas (y se crea y se agrega como parte de la operación). En caso de error: "Failed".
ResultExtentTags |string |Colección de etiquetas a la que se etiqueta la extensión del resultado (si la hay) o "null" en caso de que se produzca un error en la operación.
Detalles |string |Incluye los detalles del error, en caso de que se produzca un error en la operación.

**Ejemplos**

Quita la `drop-by:Partition000` etiqueta de cualquier extensión de `MyOtherTable` la tabla que esté etiquetada con ella.

```kusto
.drop extent tags from table MyOtherTable ('drop-by:Partition000')
```

Quita las etiquetas `drop-by:20160810104500` , `a random tag` y/o `drop-by:20160810` de cualquier extensión de la tabla `My Table` que esté etiquetada con cualquiera de ellas.

```kusto
.drop extent tags from table [My Table] ('drop-by:20160810104500','a random tag','drop-by:20160810')
```

Quita todas las `drop-by` etiquetas de las extensiones de la tabla `MyTable` .

```kusto
.drop extent tags <| 
  .show table MyTable extents 
  | where isnotempty(Tags)
  | extend Tags = split(Tags, '\r\n') 
  | mv-expand Tags to typeof(string)
  | where Tags startswith 'drop-by'
```

Quita todas las etiquetas que coinciden con regex `drop-by:StreamCreationTime_20160915(\d{6})` de las extensiones de la tabla `MyTable` .

```kusto
.drop extent tags <| 
  .show table MyTable extents 
  | where isnotempty(Tags)
  | extend Tags = split(Tags, '\r\n')
  | mv-expand Tags to typeof(string)
  | where Tags matches regex @"drop-by:StreamCreationTime_20160915(\d{6})"
```

**Salida del ejemplo** 

|OriginalExtentId |ResultExtentId | ResultExtentTags | Detalles
|---|---|---|---
|e133f050-a1e2-4dad-8552-1f5cf47cab69 |0d96ab2d-9dd2-4d2c-a45e-b24c65aa6687 | Partition001 |
|cdbeb35b-87ea-499f-b545-defbae091b57 |a90a303c-8a14-4207-8f35-d8ea94ca45be | |
|4fcb4598-9a31-4614-903c-0c67c286da8c |97aafea1-59ff-4312-b06b-08f42187872f | Partition001 Partition002 |
|2dfdef64-62a3-4950-a130-96b5b1083b5a |0fb7f3da-5e28-4f09-a000-e62eb41592df | |

## <a name="alter-extent-tags"></a>. Alter extent (etiquetas)

**Sintaxis**

`.alter`[ `async` ] `extent` `tags` `(` '*Tag1*' [ `,` '*etiqueta2*' `,` ... `,` ' *Consulta TagN*' `)`  <|  *query* ]

El comando se ejecuta en el contexto de una base de datos específica y modifica las [etiquetas de extensión](extents-overview.md#extent-tagging) de todas las extensiones devueltas por la consulta especificada, al conjunto de etiquetas proporcionado.

Las extensiones y las etiquetas que se van a modificar se especifican mediante una consulta Kusto que devuelve un conjunto de registros con una columna denominada "ExtentId".

* `async`(opcional) especifica si el comando se ejecuta de forma asincrónica (en cuyo caso, se devuelve un identificador de operación (GUID) y el estado de la operación se puede supervisar mediante el comando [. show Operations](operations.md#show-operations) ).
    * En caso de que se use esta opción, se puede recuperar el resultado de la ejecución correcta del comando [. Mostrar detalles](operations.md#show-operation-details) de la operación.

Requiere el [permiso de administrador de tablas](../management/access-control/role-based-authorization.md) para todas las tablas implicadas.

**Restricciones**
- Todas las extensiones deben estar en la base de datos de contexto y deben pertenecer a la misma tabla. 

**Devolver salida**

Parámetro de salida |Tipo |Descripción 
---|---|---
OriginalExtentId |string |Identificador único (GUID) de la extensión original cuyas etiquetas se han modificado (y se han quitado como parte de la operación). 
ResultExtentId |string |Identificador único (GUID) para la extensión del resultado que ha modificado las etiquetas (y se crea y se agrega como parte de la operación). En caso de error: "Failed".
ResultExtentTags |string |Colección de etiquetas con la que se etiqueta el resultado o "null" en caso de que se produzca un error en la operación.
Detalles |string |Incluye los detalles del error, en caso de que se produzca un error en la operación.

**Ejemplos**

Modifica las etiquetas de todas las extensiones de la tabla `MyTable` a `MyTag` .

```kusto
.alter extent tags ('MyTag') <| .show table MyTable extents
```

Modifica las etiquetas de todas las extensiones de la tabla `MyTable` , etiquetadas con `drop-by:MyTag` en `drop-by:MyNewTag` y `MyOtherNewTag` .

```kusto
.alter extent tags ('drop-by:MyNewTag','MyOtherNewTag') <| .show table MyTable extents where tags has 'drop-by:MyTag'
```

**Salida del ejemplo** 

|OriginalExtentId |ResultExtentId | ResultExtentTags | Detalles
|---|---|---|---
|e133f050-a1e2-4dad-8552-1f5cf47cab69 |0d96ab2d-9dd2-4d2c-a45e-b24c65aa6687 | Drop-by: MyNewTag MyOtherNewTag| 
|cdbeb35b-87ea-499f-b545-defbae091b57 |a90a303c-8a14-4207-8f35-d8ea94ca45be | Drop-by: MyNewTag MyOtherNewTag| 
|4fcb4598-9a31-4614-903c-0c67c286da8c |97aafea1-59ff-4312-b06b-08f42187872f | Drop-by: MyNewTag MyOtherNewTag| 
|2dfdef64-62a3-4950-a130-96b5b1083b5a |0fb7f3da-5e28-4f09-a000-e62eb41592df | Drop-by: MyNewTag MyOtherNewTag| 

