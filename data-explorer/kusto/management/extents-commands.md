---
title: 'Administración de extensiones (particiones de datos): Explorador de azure data Explorer . Microsoft Docs'
description: En este artículo se describe la administración de extensiones (particiones de datos) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: e94b830809bf079e612b477292d00d2dbe85e60e
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744725"
---
# <a name="extents-data-shards-management"></a>Gestión de extensiones (fragmentos de datos)

Las particiones de datos se denominan **extensiones** en Kusto y todos los comandos utilizan "extensión" o "extensiones" como sinónimo.

## <a name="show-extents"></a>.show extensiones

**Nivel de clúster**

`.show` `cluster` `extents` [`hot`]

Muestra información sobre las extensiones (particiones de datos) que están presentes en el clúster.
Si `hot` se especifica: muestra solo las extensiones que se espera que estén en la caché activa.

**Nivel de base de datos**

`.show``database` *DatabaseName* `extents` `(`[*ExtentId1*`,`... `,` *ExtentIdN*`)`]`hot`[`where` `tags` `has`|`contains`|`!has`|`!contains`] [`and` `tags` (`has`|`contains`|`!has`|`!contains`) *Tag1* [ ( ) *Tag2*...]]

Muestra información sobre las extensiones (particiones de datos) que están presentes en la base de datos especificada.
Si `hot` se especifica: muestra solo las extensiones que se espera que estén en la caché activa.

**Nivel de tabla**

`.show``table` *TableName* `extents` `(`[*ExtentId1*`,`... `,` *ExtentIdN*`)`]`hot`[`where` `tags` `has`|`contains`|`!has`|`!contains`] [`and` `tags` (`has`|`contains`|`!has`|`!contains`) *Tag1* [ ( ) *Tag2*...]]

`.show``tables` `,`TableName1 ... *TableName1* `(` `,` *TableNameN* `)` `extents` `(`[*ExtentId1*`,`... `,` *ExtentIdN*`)`]`hot`[`where` `tags` `has`|`contains`|`!has`|`!contains`] [`and` `tags` (`has`|`contains`|`!has`|`!contains`) *Tag1* [ ( ) *Tag2*...]]

Muestra información sobre las extensiones (particiones de datos) que están presentes en las tablas especificadas (la base de datos se toma del contexto del comando).
Si `hot` se especifica: muestra solo las extensiones que se espera que estén en la caché activa.

**NOTAS**

* El uso de comandos de nivel de `| where DatabaseName == '...' and TableName == '...'`base de datos y/o tabla es mucho más rápido que filtrar (añadir) los resultados de un comando de nivel de clúster.
* Si se proporciona la lista opcional de controladores de extensión, el conjunto de datos devuelto se limita únicamente a esas extensiones.
    * Una vez más, esto es `| where ExtentId in(...)` mucho más rápido que filtrar (añadir a) los resultados de los comandos "bare".
* En `tags` caso de que se especifiquen filtros:
  * La lista devuelta está limitada a las extensiones cuya colección de etiquetas obedece a *todos los* filtros de etiquetas proporcionados.
    * Una vez más, esto es `| where Tags has '...' and Tags contains '...'`mucho más rápido que filtrar (añadir) los resultados de los comandos "bare".
  * `has`los filtros son filtros de igualdad: las extensiones que no están etiquetadas con ninguna de las etiquetas especificadas se filtrarán.
  * `!has`los filtros son filtros negativos de igualdad: las extensiones que se etiquetan con cualquiera de las etiquetas especificadas se filtrarán.
  * `contains`los filtros son filtros de subcadena que no distinguen mayúsculas de minúsculas: las extensiones que no tienen las cadenas especificadas como subcadena de cualquiera de sus etiquetas se filtrarán. 
  * `!contains`los filtros son filtros negativos de subcadena que no distinguen mayúsculas de minúsculas: las extensiones que tienen las cadenas especificadas como una subcadena de cualquiera de sus etiquetas se filtrarán. 
  
  * **Ejemplos**
    * Extensión `E` en `T` la tabla `aaa` `BBB` se `ccc`etiqueta con etiquetas y .
    * Esta consulta `E`devolverá: 
    
    ```kusto 
    .show table T extents where tags has 'aaa' and tags contains 'bb' 
    ``` 
    
    * Esta consulta *not* no `E` volverá (ya que no está `aa`etiquetada con una etiqueta que es igual a):
    
    ```kusto 
    .show table T extents where tags has 'aa' and tags contains 'bb' 
    ``` 
    
    * Esta consulta `E`devolverá: 
    
    ```kusto 
    .show table T extents where tags contains 'aaa' and tags contains 'bb' 
    ``` 

|Parámetro de salida |Tipo |Descripción 
|---|---|---
|ExtentId |Guid |Identificación de la extensión 
|DatabaseName |String |Base de datos que la extensión pertenece a 
|TableName |String |Cuadro de que las extensiones pertenecen a 
|MaxCreatedOn |DateTime |Fecha y hora en que se creó la extensión (para una extensión combinada - el máximo de tiempos de creación entre las extensiones de origen) 
|OriginalSize |Double |Tamaño original en bytes de los datos de extensión 
|ExtentSize |Double |Tamaño de la extensión en la memoria (comprimido + índice) 
|CompressedSize |Double |Tamaño comprimido de los datos de extensión en la memoria 
|IndexSize |Double |Tamaño del índice de los datos de extensión 
|Blocks |long |Cantidad de bloques de datos en extensión 
|Segmentos |long |Cantidad de segmento de datos en extensión 
|AssignedDataNodes |String | En desuso (una cadena vacía)
|LoadedDataNodes |String |En desuso (una cadena vacía)
|ExtentContainerId |String | Identificación de la extensión del contenedor en la medida en que se
|RowCount |long |Cantidad de filas en la medida en que
|MinCreatedOn |DateTime |Fecha y hora en que se creó la extensión (para una extensión combinada - el mínimo de tiempos de creación entre las extensiones de origen) 
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
 
## <a name="merge-extents"></a>Extensiones .merge

**Sintaxis**

`.merge``[async | dryrun]` *NombreDeTabla* `(` *GUID1* [`,` *GUID2* ...] `)``[with(rebuild=true)]`

Este comando combina las extensiones (Ver: Información general sobre [extensiones (particiones de datos))](extents-overview.md)indicadas por sus iDs en la tabla especificada.

Hay 2 tipos para las operaciones de combinación: *Combinar* (que reconstruye índices) y *Reconstruir* (que recupera completamente los datos).

* `async`: la operación se realizará de forma asincrónica: el resultado será `.show operations <operation ID>` un identificador de operación (GUID), que se puede ejecutar con, para ver el estado del comando & estado.
* `dryrun`: la operación solo enumerará las extensiones que se deben combinar, pero que en realidad no las combinarán. 
* `with(rebuild=true)`: las extensiones se reconstruirán (los datos se recuperarán) en lugar de fusionarse (los índices se reconstruirán).

**Salida de retorno**

Parámetro de salida |Tipo |Descripción 
---|---|---
OriginalExtentId |string |Identificador único (GUID) para la extensión original de la tabla de origen, que se ha combinado en la extensión de destino. 
ResultExtentId |string |Identificador único (GUID) para la extensión de resultado que se ha creado a partir de las extensiones de origen. Tras el error - "Fallido" o "Abandoned".
Duration |timespan |El período de tiempo que se tardó en completar la operación de combinación.

**Ejemplos**

Reconstruye dos extensiones `MyTable`específicas en , realizadas de forma asincrónica
```kusto
.merge async MyTable (e133f050-a1e2-4dad-8552-1f5cf47cab69, 0d96ab2d-9dd2-4d2c-a45e-b24c65aa6687) with(rebuild=true)
```

Combina dos extensiones `MyTable`específicas en , realizado sincrónicamente
```kusto
.merge MyTable (12345050-a1e2-4dad-8552-1f5cf47cab69, 98765b2d-9dd2-4d2c-a45e-b24c65aa6687)
```

**Notas**
- En General, `.merge` los comandos rara vez se deben ejecutar manualmente y se realizan continuamente automáticamente en segundo plano del clúster de Kusto (según las directivas de combinación definidas para [tablas](mergepolicy.md) y bases de datos en él).  
  - Las consideraciones generales sobre los criterios para fusionar varias extensiones en una sola se documentan en Directiva de [mezcla](mergepolicy.md).
- `.merge`las operaciones tienen `Abandoned` un posible estado final `.show operations` de (que se puede ver cuando se ejecuta con el identificador de operación): este estado sugiere que las extensiones de origen no se fusionaron, ya que algunas de las extensiones de origen ya no existen en la tabla de origen.
  - Se espera que tal estado ocurra en casos tales como:
     - Las extensiones de origen se han descartado como parte de la retención de la tabla.
     - Las extensiones de origen se han movido a una tabla diferente.
     - La tabla de origen se ha quitado o cambiado de nombre por completo.

## <a name="move-extents"></a>.move extensions

**Sintaxis**

`.move`[`async` `extents` `all` `from` ] `table` *SourceTableName* `to` `table` *DestinationTableName*

`.move`[`async` `extents` `(` ] *GUID1* [`,` *GUID2* ...] `)` `table` *DestinationTableName* *SourceTableName* SourceTableName `to` DestinationTableName `from` `table` 

`.move`[`async` `extents` `to` `table` ] *Consulta DestinationTableName* <| *query*

Este comando se ejecuta en el contexto de una base de datos específica y mueve las extensiones especificadas de la tabla de origen a la tabla de destino transaccionalmente.
Requiere [permiso](../management/access-control/role-based-authorization.md) de administrador de tabla para las tablas de origen y destino.

* `async`(opcional) especifica si el comando se ejecuta de forma asincrónica (en cuyo caso se devuelve un identificador de operación (Guid) y se puede supervisar el estado de la operación mediante el comando [.show operations).](operations.md#show-operations)
    * En caso de que se utilice esta opción, los resultados de una ejecución correcta se pueden recuperar el comando [.show operation details).](operations.md#show-operation-details)

Hay tres maneras de especificar qué extensiones se mueven:
1. Todas las extensiones de una tabla específica deben moverse.
2. Especificando explícitamente los identificadores de extensión en la tabla de origen.
3. Proporcionando una consulta cuyos resultados especifican la extensión de los identificadores en las tablas de origen.

**Restricciones**
- Las tablas de origen y de destino deben estar en la base de datos de contexto. 
- Se espera que todas las columnas de la tabla de origen existan en la tabla de destino con el mismo nombre y tipo de datos.

**Especificación de extensiones con una consulta**

```kusto 
.move extents to table TableName <| ...query... 
```

Las extensiones se especifican mediante una consulta Kusto que devuelve un conjunto de registros con una columna denominada "ExtentId". 

**Salida de retorno** (para la ejecución de sincronización)

Parámetro de salida |Tipo |Descripción 
---|---|---
OriginalExtentId |string |Identificador único (GUID) para la extensión original de la tabla de origen, que se ha movido a la tabla de destino. 
ResultExtentId |string |Identificador único (GUID) para la extensión de resultado que se ha movido de la tabla de origen a la tabla de destino. Tras el error - "Fallido".
Detalles |string |Incluye los detalles del error, en caso de que se produzca un error en la operación.

**Ejemplos**

Mueve todas las `MyTable` extensiones `MyOtherTable`de la tabla a la tabla.
```kusto
.move extents all from table MyTable to table MyOtherTable
```

Mueve 2 extensiones específicas (por sus `MyTable` ideos de extensión) de tabla a tabla. `MyOtherTable`
```kusto
.move extents (AE6CD250-BE62-4978-90F2-5CB7A10D16D7,399F9254-4751-49E3-8192-C1CA78020706) from table MyTable to table MyOtherTable
```

Mueve todas las extensiones`MyTable1`de `MyTable2`2 `MyOtherTable`tablas específicas ( , ) a la tabla .
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

## <a name="drop-extents"></a>.drop extensiones

Quita las extensiones de la base de datos /tabla especificada. Este comando tiene varias variantes: en una variante, las extensiones que se van a quitar se especifican mediante una consulta Kusto, mientras que en las otras extensiones de variantes se especifican mediante un minilenguaje que se describe a continuación. 
 
### <a name="specifying-extents-with-a-query"></a>Especificación de extensiones con una consulta

Requiere [permiso](../management/access-control/role-based-authorization.md) de administrador de tabla para cada una de las tablas que tienen extensiones devueltas por la consulta proporcionada.

Elimina extensiones (o simplemente las `whatif` notifica sin realmente descartar si se utiliza):

**Sintaxis**

`.drop``extents` [`whatif`] <? *consulta*

Las extensiones se especifican mediante una consulta Kusto que devuelve un conjunto de registros con una columna denominada "ExtentId". 
 
### <a name="dropping-a-specific-extent"></a>Eliminación de una extensión específica

Requiere [el permiso de administrador de tabla](../management/access-control/role-based-authorization.md) en caso de que se especifique el nombre de la tabla.

Requiere [permiso](../management/access-control/role-based-authorization.md) de administrador de base de datos en caso de que no se especifique el nombre de la tabla.

**Sintaxis**

`.drop``extent` *ExtentId* `from` [ *TableName*]

### <a name="dropping-specific-multiple-extents"></a>Eliminación de múltiples extensiones específicas

Requiere [el permiso de administrador de tabla](../management/access-control/role-based-authorization.md) en caso de que se especifique el nombre de la tabla.

Requiere [permiso](../management/access-control/role-based-authorization.md) de administrador de base de datos en caso de que no se especifique el nombre de la tabla.

**Sintaxis**

`.drop``extents` `,`ExtentId1 ... *ExtentId1* `(` *ExtentIdN* `)` `from` [ *NombreDeTabla*]

### <a name="specifying-extents-by-properties"></a>Especificación de extensiones por propiedades

Requiere [el permiso de administrador de tabla](../management/access-control/role-based-authorization.md) en caso de que se especifique el nombre de la tabla.

Requiere [permiso](../management/access-control/role-based-authorization.md) de administrador de base de datos en caso de que no se especifique el nombre de la tabla.

`.drop``extents` [`older` *N* N`days` | `hours`( `from` )] (`trim` `by` `extentsize` | *TableName*  |  `all` `tables`) [`limit` (`datasize`) *N* (`MB` | `GB` | `bytes`)] [ *LimitCount*]

* `older`: Solo se eliminarán las extensiones anteriores a *N* días/horas. 
* `trim`: La operación recortará los datos de la base de datos hasta que la suma de extensiones coincida con el tamaño deseado (MaxSize) 
* `limit`: La operación se aplicará a las primeras extensiones *LimitCount* 

El comando admite`.drop-pretend` el `.drop`modo de emulación (en lugar de ), que produce una salida como si el comando se hubiera ejecutado, pero sin ejecutarlo realmente.

**Ejemplos**

Elimine todas las extensiones creadas hace más de 10 días de todas las tablas de la base de datos`MyDatabase`

```kusto
.drop extents <| .show database MyDatabase extents | where CreatedOn < now() - time(10d)
```

Elimina todas las extensiones de las tablas `Table1` y `Table2`, cuyo tiempo de creación fue de hace más de 10 días:

```kusto
.drop extents older 10 days from tables (Table1, Table2)
```

Modo de emulación: muestra qué extensiones serían quitadas por el comando (el parámetro ID de extensión no es aplicable para este comando):

```kusto
.drop-pretend extents older 10 days from all tables
```

Elimina todas las extensiones de 'TestTable':

```kusto
.drop extents from TestTable
```
 
**Salida de retorno**

|Parámetro de salida |Tipo |Descripción 
|---|---|---
|ExtentId |String |ExtentId que se ha quitado como resultado del comando 
|TableName |String |Nombre de la tabla, donde pertenecía la extensión  
|CreatedOn |DateTime |Marca de tiempo que contiene información sobre cuándo se creó inicialmente la extensión 
 
**Salida del ejemplo** 

|Extensión De N.o de la |Nombre de la tabla |Creada el 
|---|---|---
|43c6e03f-1713-4ca7-a52a-5db8a4e8b87d |TestTable |2015-01-12 12:48:49.4298178 

## <a name="replace-extents"></a>.reemplazar extensiones

**Sintaxis**

`.replace`[`async` `extents` `in` `table` ] *Consulta DestinationTableName* `<| 
{` *para que se quiten las extensiones de la*`},{`consulta de tabla*para que las extensiones se muevan a la tabla*`}`

Este comando se ejecuta en el contexto de una base de datos específica, mueve las extensiones especificadas de sus tablas de origen a la tabla de destino y quita las extensiones especificadas de la tabla de destino.
Todas las operaciones de colocación y movimiento se realizan en una sola transacción.

Requiere [permiso](../management/access-control/role-based-authorization.md) de administrador de tabla para las tablas de origen y destino.

* `async`(opcional) especifica si el comando se ejecuta de forma asincrónica (en cuyo caso se devuelve un identificador de operación (Guid) y se puede supervisar el estado de la operación mediante el comando [.show operations).](operations.md#show-operations)
    * En caso de que se utilice esta opción, los resultados de una ejecución correcta se pueden recuperar el comando [.show operation details).](operations.md#show-operation-details)

Especificar qué extensiones se deben quitar o mover se realiza proporcionando 2 consultas
- *consulta de las extensiones que se quitarán de* la tabla: los resultados de esta consulta especifican la extensión de los identificadores  
que se debe quitar de la tabla de destino.
- *consulta de extensiones que se van a mover a* la tabla: los resultados de esta consulta especifican los identificadores de extensión de las tablas de origen que se deben mover a la tabla de destino.

Ambas consultas deben devolver un conjunto de registros con una columna denominada "ExtentId".

**Restricciones**
- Las tablas de origen y de destino deben estar en la base de datos de contexto. 
- Se espera que todas las extensiones especificadas por la *consulta para que las extensiones se quiten de* la tabla pertenezcan a la tabla de destino.
- Se espera que todas las columnas de las tablas de origen existan en la tabla de destino con el mismo nombre y tipo de datos.

**Salida de retorno** (para la ejecución de sincronización)

Parámetro de salida |Tipo |Descripción 
---|---|---
OriginalExtentId |string |Identificador único (GUID) para la extensión original de la tabla de origen, que se ha movido a la tabla de destino o - la extensión de la tabla de destino, que se ha quitado.
ResultExtentId |string |Identificador único (GUID) para la extensión de resultado que se ha movido de la tabla de origen a la tabla de destino, o - vacío, en caso de que la extensión se quitó de la tabla de destino. Tras el error - "Fallido".
Detalles |string |Incluye los detalles del error, en caso de que se produzca un error en la operación.

**Ejemplos**

Mueve todas las extensiones`MyTable1`de `MyTable2`2 `MyOtherTable`tablas específicas ( `MyOtherTable` , `drop-by:MyTag`) a la tabla , y quita todas las extensiones en etiquetadas con:

```kusto
.replace extents in table MyOtherTable <|
    {.show table MyOtherTable extents where tags has 'drop-by:MyTag'},
    {.show tables (MyTable1,MyTable2) extents}
```

**Salida del ejemplo** 

|OriginalExtentId |ResultExtentId |Detalles
|---|---|---
|e133f050-a1e2-4dad-8552-1f5cf47cab69 |0d96ab2d-9dd2-4d2c-a45e-b24c65aa6687| 
|cdbeb35b-87ea-499f-b545-defbae091b57 |a90a303c-8a14-4207-8f35-d8ea94ca45be| 
|4fcb4598-9a31-4614-903c-0c67c286da8c |97aafea1-59ff-4312-b06b-08f42187872f| 
|2dfdef64-62a3-4950-a130-96b5b1083b5a |0fb7f3da-5e28-4f09-a000-e62eb41592df| 

*NOTA*: 
> [!NOTE]
> El comando producirá un error si las extensiones devueltas por las *extensiones que se van a quitar de* la consulta de tabla no existen en la tabla de destino. Esto puede suceder si las extensiones se fusionaron antes de ejecutar el comando replace. Para asegurarse de que el comando produce un error en las extensiones que faltan, compruebe que la consulta devuelve los ExtensionId esperados. Ejemplo #1 siguiente producirá un error si la extensión de colocación no existe en la tabla MyOtherTable. Ejemplo #2, sin embargo, se realizará correctamente aunque la extensión de colocación no existe, ya que la consulta que se va a quitar no devolvió ningún identificador de extensión. 

Ejemplo 1: 

```kusto
.replace extents in table MyOtherTable <|
     { datatable(ExtentId:guid)[ "2cca5844-8f0d-454e-bdad-299e978be5df"] }, { .show table MyTable1 extents }
```

Ejemplo 2:

```kusto
.replace extents in table MyOtherTable  <|
     { .show table MyOtherTable extents | where ExtentId == guid(2cca5844-8f0d-454e-bdad-299e978be5df) }, { .show table MyTable1 extents }
```



## <a name="drop-extent-tags"></a>Etiquetas de extensión .drop

**Sintaxis**

`.drop`[`async` `extent` `tags` `from` ] `table` *NombreTabla* `(`'`,`*Tag1*'[ '*Tag2*'`,`... `,`'*TagN*']`)`

`.drop`[`async` `extent` `tags`  <| ] *consulta*

* `async`(opcional) especifica si el comando se ejecuta de forma asincrónica (en cuyo caso se devuelve un identificador de operación (Guid) y se puede supervisar el estado de la operación mediante el comando [.show operations).](operations.md#show-operations)
    * En caso de que se utilice esta opción, los resultados de una ejecución correcta se pueden recuperar el comando [.show operation details).](operations.md#show-operation-details)

El comando se ejecuta en el contexto de una base de datos específica y quita las etiquetas de [extensión](extents-overview.md#extent-tagging) proporcionadas de cualquier extensión en la base de datos y la tabla proporcionadas, que incluye cualquiera de las etiquetas.  

Hay dos maneras de especificar qué etiquetas se deben quitar de qué extensiones:

1. Especificando explícitamente las etiquetas que se deben quitar de todas las extensiones de la tabla especificada.
2. Proporcionando una consulta cuyos resultados especifican la extensión de identificadores en la tabla y foreach extent - las etiquetas que se deben quitar.

**Restricciones**
- Todas las extensiones deben estar en la base de datos de contexto y deben pertenecer a la misma tabla. 

**Especificación de extensiones con una consulta**

Requiere [permiso](../management/access-control/role-based-authorization.md) de administrador de tabla para todas las tablas de origen y destino implicadas.

```kusto 
.drop extent tags <| ...query... 
```

Las extensiones y las etiquetas que se van a quitar se especifican mediante una consulta Kusto que devuelve un conjunto de registros con una columna denominada "ExtentId" y una columna denominada "Tags". 

*NOTA*: Al utilizar la biblioteca de [cliente Kusto .NET](../api/netfx/about-kusto-data.md), puede utilizar los métodos siguientes para generar el comando deseado:
- `CslCommandGenerator.GenerateExtentTagsDropByRegexCommand(string tableName, string regex)`
- `CslCommandGenerator.GenerateExtentTagsDropBySubstringCommand(string tableName, string substring)`

**Salida de retorno**

Parámetro de salida |Tipo |Descripción 
---|---|---
OriginalExtentId |string |Identificador único (GUID) para la extensión original cuyas etiquetas se han modificado (y se quita como parte de la operación) 
ResultExtentId |string |Identificador único (GUID) para la extensión de resultado que tiene etiquetas modificadas (y se crea y se agrega como parte de la operación). Tras el error - "Fallido".
ResultExtentTags |string |La colección de etiquetas con las que se etiqueta la extensión de resultado (si la hay) o "null" en caso de que se produzca un error en la operación.
Detalles |string |Incluye los detalles del error, en caso de que se produzca un error en la operación.

**Ejemplos**

Quita `drop-by:Partition000` la etiqueta de `MyOtherTable` cualquier extensión de la tabla que esté etiquetada con ella.

```kusto
.drop extent tags from table MyOtherTable ('drop-by:Partition000')
```

Quita las `drop-by:20160810104500` `a random tag`etiquetas , `drop-by:20160810` , y/o `My Table` desde cualquier extensión en la tabla que está etiquetada con cualquiera de ellas.

```kusto
.drop extent tags from table [My Table] ('drop-by:20160810104500','a random tag','drop-by:20160810')
```

Quita `drop-by` todas las etiquetas `MyTable`de las extensiones de la tabla.

```kusto
.drop extent tags <| 
  .show table MyTable extents 
  | where isnotempty(Tags)
  | extend Tags = split(Tags, '\r\n') 
  | mv-expand Tags to typeof(string)
  | where Tags startswith 'drop-by'
```

Quita todas las `drop-by:StreamCreationTime_20160915(\d{6})` etiquetas que coinciden `MyTable`con regex de extensiones en la tabla .

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

## <a name="alter-extent-tags"></a>Etiquetas de extensión .alter

**Sintaxis**

`.alter`[`async` `extent` `tags` `(`] '*Tag1*`,`'[`,`'*Tag2*' ... `,`'*Consulta TagN*`)` <| *']*

El comando se ejecuta en el contexto de una base de datos específica y modifica las etiquetas de [extensión](extents-overview.md#extent-tagging) de todas las extensiones devueltas por la consulta especificada, al conjunto proporcionado de etiquetas.

Las extensiones y las etiquetas que se van a modificar se especifican mediante una consulta Kusto que devuelve un conjunto de registros con una columna denominada "ExtentId".

* `async`(opcional) especifica si el comando se ejecuta de forma asincrónica (en cuyo caso se devuelve un identificador de operación (Guid) y se puede supervisar el estado de la operación mediante el comando [.show operations).](operations.md#show-operations)
    * En caso de que se utilice esta opción, los resultados de una ejecución correcta se pueden recuperar el comando [.show operation details).](operations.md#show-operation-details)

Requiere [permiso](../management/access-control/role-based-authorization.md) de administrador de tabla para todas las tablas implicadas.

**Restricciones**
- Todas las extensiones deben estar en la base de datos de contexto y deben pertenecer a la misma tabla. 

**Salida de retorno**

Parámetro de salida |Tipo |Descripción 
---|---|---
OriginalExtentId |string |Identificador único (GUID) para la extensión original cuyas etiquetas se han modificado (y se quita como parte de la operación) 
ResultExtentId |string |Identificador único (GUID) para la extensión de resultado que tiene etiquetas modificadas (y se crea y se agrega como parte de la operación). Tras el error - "Fallido".
ResultExtentTags |string |La colección de etiquetas con las que se etiqueta la extensión de resultado, o "null" en caso de que se produzca un error en la operación.
Detalles |string |Incluye los detalles del error, en caso de que se produzca un error en la operación.

**Ejemplos**

Modifica las etiquetas de todas `MyTable` las `MyTag`extensiones de la tabla a .

```kusto
.alter extent tags ('MyTag') <| .show table MyTable extents
```

Altera las etiquetas de todas `MyTable`las extensiones de la tabla , etiquetadas con `drop-by:MyTag` to `drop-by:MyNewTag` y `MyOtherNewTag`.

```kusto
.alter extent tags ('drop-by:MyNewTag','MyOtherNewTag') <| .show table MyTable extents where tags has 'drop-by:MyTag'
```

**Salida del ejemplo** 

|OriginalExtentId |ResultExtentId | ResultExtentTags | Detalles
|---|---|---|---
|e133f050-a1e2-4dad-8552-1f5cf47cab69 |0d96ab2d-9dd2-4d2c-a45e-b24c65aa6687 | drop-by:MyNewTag MyOtherNewTag| 
|cdbeb35b-87ea-499f-b545-defbae091b57 |a90a303c-8a14-4207-8f35-d8ea94ca45be | drop-by:MyNewTag MyOtherNewTag| 
|4fcb4598-9a31-4614-903c-0c67c286da8c |97aafea1-59ff-4312-b06b-08f42187872f | drop-by:MyNewTag MyOtherNewTag| 
|2dfdef64-62a3-4950-a130-96b5b1083b5a |0fb7f3da-5e28-4f09-a000-e62eb41592df | drop-by:MyNewTag MyOtherNewTag| 

