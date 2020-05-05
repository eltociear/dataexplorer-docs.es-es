---
title: 'Administración de tablas externas: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la administración de tablas externas en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: c52f0649531678e31310f5a1f4bfb97f99f15857
ms.sourcegitcommit: 4f68d6dbfa6463dbb284de0aa17fc193d529ce3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82741941"
---
# <a name="external-table-management"></a>Administración de tablas externas

Vea [tablas externas](../query/schema-entities/externaltables.md) para obtener información general sobre las tablas externas. 

## <a name="common-external-tables-control-commands"></a>Comandos de control comunes de tablas externas

Los siguientes comandos son relevantes para _cualquier_ tabla externa (de cualquier tipo).

### <a name="show-external-tables"></a>. Mostrar tablas externas

* Devuelve todas las tablas externas de la base de datos (o una tabla externa específica).
* Requiere el [permiso monitor de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis:** 

`.show` `external` `tables`

`.show``external` *TableName* TableName `table`

**Salida**

| Parámetro de salida | Tipo   | Descripción                                                         |
|------------------|--------|---------------------------------------------------------------------|
| TableName        | cadena | Nombre de la tabla externa                                             |
| TableType        | cadena | Tipo de tabla externa                                              |
| Carpeta           | cadena | Carpeta de la tabla                                                     |
| DocString        | cadena | Cadena que documenta la tabla                                       |
| Propiedades       | cadena | Propiedades serializadas de JSON de la tabla (específicas del tipo de tabla) |


**Ejemplos:**

```kusto
.show external tables
.show external table T
```

| TableName | TableType | Carpeta         | DocString | Propiedades |
|-----------|-----------|----------------|-----------|------------|
| T         | Blob      | ExternalTables | Docs      | {}         |


### <a name="show-external-table-schema"></a>. Mostrar esquema de tabla externa

* Devuelve el esquema de la tabla externa, como JSON o CSL. 
* Requiere el [permiso monitor de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis:** 

`.show``external` `schema` *TableName* (`json`) `table` `as`  | `csl`

`.show``external` *TableName* TableName `table``cslschema`

**Salida**

| Parámetro de salida | Tipo   | Descripción                        |
|------------------|--------|------------------------------------|
| TableName        | cadena | Nombre de la tabla externa            |
| Schema           | cadena | El esquema de tabla en formato JSON |
| DatabaseName     | cadena | Nombre de la base de datos de la tabla             |
| Carpeta           | cadena | Carpeta de la tabla                    |
| DocString        | cadena | Cadena que documenta la tabla      |

**Ejemplos:**

```kusto
.show external table T schema as JSON
```

```kusto
.show external table T schema as CSL
.show external table T cslschema
```

**Salida:**

*JSON*

| TableName | Schema    | DatabaseName | Carpeta         | DocString |
|-----------|----------------------------------|--------------|----------------|-----------|
| T         | {"Name": "ExternalBlob",<br>"Carpeta": "ExternalTables",<br>"DocString": "docs",<br>"OrderedColumns": [{"Name": "x", "type": "System. Int64", "CslType": "Long", "DocString": ""}, {"Name": "s", "type": "System. String", "CslType": "String", "DocString": ""}]} | DB           | ExternalTables | Docs      |


*CSL*

| TableName | Schema          | DatabaseName | Carpeta         | DocString |
|-----------|-----------------|--------------|----------------|-----------|
| T         | x:long,s:string | DB           | ExternalTables | Docs      |

### <a name="drop-external-table"></a>. quitar tabla externa

* Quita una tabla externa 
* No se puede restaurar la definición de la tabla externa después de esta operación
* Requiere [permiso de administrador de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis:**  

`.drop``external` *TableName* TableName `table`

**Salida**

Devuelve las propiedades de la tabla quitada. Vea [. Mostrar tablas externas](#show-external-tables).

**Ejemplos:**

```kusto
.drop external table ExternalBlob
```

| TableName | TableType | Carpeta         | DocString | Schema       | Propiedades |
|-----------|-----------|----------------|-----------|-----------------------------------------------------|------------|
| T         | Blob      | ExternalTables | Docs      | [{"Name": "x", "CslType": "Long"},<br> {"Name": "s", "CslType": "String"}] | {}         |

## <a name="external-tables-in-azure-storage-or-azure-data-lake"></a>Tablas externas en Azure Storage o Azure Data Lake

El siguiente comando describe cómo crear una tabla externa. La tabla se puede encontrar en Azure Blob Storage, Azure Data Lake Store GEN1 o Azure Data Lake Store de la segunda generación. 
[Cadenas de conexión de almacenamiento](../api/connection-strings/storage.md) describe la creación de la cadena de conexión para cada una de estas opciones. 

### <a name="create-or-alter-external-table"></a>. Create o. Alter external Table

**Sintaxis**

`.create` | (`.alter`) `external` *TableName (* *esquema*) `table`  
`kind` `=` (`blob` | `adl`)  
[`partition` `by` *Partition* [`,` ....]]  
`dataformat``=` *Formato*  
`(`  
*StorageConnectionString* [`,` ...]  
`)`  
[`with` `(`[`docstring` `=` *FolderName* `=` *value* *Documentation* `folder` `,` *property_name* Documentación] [`,` NombreCarpeta], property_name valor... `=` `)`]

Crea o modifica una nueva tabla externa en la base de datos en la que se ejecuta el comando.

**Parámetros**

* *TableName* : nombre de tabla externa. Debe seguir las reglas para [los nombres de entidad](../query/schema-entities/entity-names.md). Una tabla externa no puede tener el mismo nombre que una tabla normal de la misma base de datos.
* Esquema *-esquema de datos* externos en formato `ColumnName:ColumnType[, ColumnName:ColumnType ...]`:. Si no se conoce el esquema de datos externos, use el complemento de [infer_storage_schema](../query/inferstorageschemaplugin.md) , que puede deducir el esquema en función del contenido del archivo externo.
* *Partición* : una o varias definiciones de partición (opcional). Vea la sintaxis de partición a continuación.
* *Format* : formato de datos. Se admite cualquiera de los [formatos de ingesta](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats) para realizar consultas. El uso de una tabla externa para el [escenario de exportación](data-export/export-data-to-an-external-table.md) se limita `CSV`a `TSV`los `JSON`siguientes `Parquet`formatos:,,,.
* *StorageConnectionString* : una o varias rutas de acceso para Azure BLOB Storage contenedores de blobs o sistemas de archivos Azure Data Lake Store (directorios virtuales o carpetas), incluidas las credenciales. Consulte [cadenas de conexión de almacenamiento](../api/connection-strings/storage.md) para obtener más información. Es muy recomendable proporcionar más de una cuenta de almacenamiento para evitar la limitación de almacenamiento si se [exportan](data-export/export-data-to-an-external-table.md) grandes cantidades de datos a la tabla externa. La exportación distribuirá las escrituras entre todas las cuentas proporcionadas. 

**Sintaxis de partición**

[`format_datetime =` *DateTimePartitionFormat*] `bin(` *TimestampColumnName*, *PartitionByTimeSpan*`)`  
|   
[*StringFormatPrefix*] *StringColumnName* [*StringFormatSuffix*])

**Parámetros de partición**

* *DateTimePartitionFormat* : el formato de la estructura de directorios necesaria en la ruta de acceso de salida (opcional). Si se define la creación de particiones y el formato no se especifica, el valor predeterminado es "AAAA/MM/DD/HH/mm", en función de PartitionByTimeSpan. Por ejemplo, si crea una partición por 1D, Structure será "AAAA/MM/DD". Si crea particiones por 1H, Structure será "AAAA/MM/DD/HH".
* *TimestampColumnName* : columna de fecha y hora en la que se crean las particiones de la tabla. La columna de marca de tiempo debe existir en la definición de esquema de tabla externa y en la salida de la consulta de exportación al exportar a la tabla externa.
* *PartitionByTimeSpan* : literal de TimeSpan por el que se va a particionar.
* *StringFormatPrefix* : un literal de cadena constante que formará parte de la ruta de acceso del artefacto, concatenado antes del valor de la tabla (opcional).
* *StringFormatSuffix* : un literal de cadena constante que formará parte de la ruta de acceso del artefacto, concatenado después del valor de la tabla (opcional).
* *StringColumnName* : columna de cadena en la que se crean las particiones de la tabla. La columna de cadena debe existir en la definición de esquema de tabla externa.

**Propiedades opcionales**:

| Propiedad.         | Tipo     | Descripción       |
|------------------|----------|-------------------------------------------------------------------------------------|
| `folder`         | `string` | Carpeta de la tabla                                                                     |
| `docString`      | `string` | Cadena que documenta la tabla                                                       |
| `compressed`     | `bool`   | Si se establece, indica si los blobs se `.gz` comprimen como archivos                  |
| `includeHeaders` | `string` | En el caso de los blobs CSV o TSV, indica si los blobs contienen un encabezado                     |
| `namePrefix`     | `string` | Si se establece, indica el prefijo de los BLOBs. En las operaciones de escritura, todos los blobs se escribirán con este prefijo. En las operaciones de lectura, solo se leen los blobs con este prefijo. |
| `fileExtension`  | `string` | Si se establece, indica extensiones de archivo de los BLOBs. Al escribir, los nombres de los blobs terminarán con este sufijo. En la lectura, solo se leerán los blobs con esta extensión de archivo.           |
| `encoding`       | `string` | Indica cómo se codifica el texto: `UTF8NoBOM` (valor predeterminado) o. `UTF8BOM`             |

Para obtener más información sobre los parámetros de tabla externa en las consultas, vea [lógica de filtrado de artefactos](#artifact-filtering-logic).

> [!NOTE]
> * Si la tabla existe, `.create` se producirá un error en el comando. Se `.alter` usa para modificar las tablas existentes. 
> * No se admite la modificación del esquema, el formato o la definición de partición de una tabla de blobs externos. 

Requiere [permiso de usuario](../management/access-control/role-based-authorization.md) de `.create` base de datos para y `.alter`permiso de administrador de [tablas](../management/access-control/role-based-authorization.md) para. 
 
**Ejemplo** 

Una tabla externa sin particiones. Se espera que todos los artefactos estén directamente en los contenedores definidos:

```kusto
.create external table ExternalBlob (x:long, s:string) 
kind=blob
dataformat=csv
( 
   h@'https://storageaccount.blob.core.windows.net/container1;secretKey'
)
with 
(
   docstring = "Docs",
   folder = "ExternalTables"
)  
```

Una tabla externa con particiones por fecha y hora. Los artefactos residen en directorios con el formato "AAAA/MM/DD" en las rutas de acceso definidas:

```kusto
.create external table ExternalAdlGen2 (Timestamp:datetime, x:long, s:string) 
kind=adl
partition by bin(Timestamp, 1d)
dataformat=csv
( 
   h@'abfss://filesystem@storageaccount.dfs.core.windows.net/path;secretKey'
)
with 
(
   docstring = "Docs",
   folder = "ExternalTables"
)  
```

Una tabla externa con particiones por fecha y hora con un formato de directorio "Year = YYYY/month = MM/Day = DD":

```kusto
.create external table ExternalPartitionedBlob (Timestamp:datetime, x:long, s:string) 
kind=blob
partition by format_datetime="'year='yyyy/'month='MM/'day='dd" bin(Timestamp, 1d)
dataformat=csv
( 
   h@'https://storageaccount.blob.core.windows.net/container1;secretKey'
)
with 
(
   docstring = "Docs",
   folder = "ExternalTables"
)
```

Una tabla externa con particiones de datos mensuales y un formato de directorio de "AAAA/MM":

```kusto
.create external table ExternalPartitionedBlob (Timestamp:datetime, x:long, s:string) 
kind=blob
partition by format_datetime="yyyy/MM" bin(Timestamp, 1d)
dataformat=csv
( 
   h@'https://storageaccount.blob.core.windows.net/container1;secretKey'
)
with 
(
   docstring = "Docs",
   folder = "ExternalTables"
)
```

Una tabla externa con dos particiones. La estructura de directorios es la concatenación de ambas particiones: CustomerName con formato seguido de formato de fecha y hora predeterminado. Por ejemplo, "CustomerName = Softworks/2011/11/11":

```kusto
.create external table ExternalMultiplePartitions (Timestamp:datetime, CustomerName:string) 
kind=blob
partition by 
   "CustomerName="CustomerName,
   bin(Timestamp, 1d)
dataformat=csv
( 
   h@'https://storageaccount.blob.core.windows.net/container1;secretKey'
)
with 
(
   docstring = "Docs",
   folder = "ExternalTables"   
)
```

**Salida**

|TableName|TableType|Carpeta|DocString|Propiedades|ConnectionStrings|Particiones|
|---|---|---|---|---|---|---|
|ExternalMultiplePartitions|Blob|ExternalTables|Docs|{"Format": "CSV", "Compressed": false, "CompressionType": null, "FileExtension": "CSV", "IncludeHeaders": "none", "Encoding": null, "NamePrefix": null}|["https://storageaccount.blob.core.windows.net/container1;*******"]}|[{"StringFormat": "CustomerName ={0}", "ColumnName": "customername", "ordinal": 0}, PartitionBy ":" 1.00:00:00 "," ColumnName ":" timestamp "," ordinal ": 1}]|

#### <a name="artifact-filtering-logic"></a>Lógica de filtrado de artefactos

Al consultar una tabla externa, el motor de consultas filtra los artefactos de almacenamiento externo irrelevantes (BLOB) para mejorar el rendimiento de las consultas. A continuación se describe el proceso de iteración en blobs y la determinación de si se debe procesar un BLOB.

1. Cree un patrón de URI que represente un lugar donde se encuentren los BLOBs. Inicialmente, el patrón de URI es igual a una cadena de conexión proporcionada como parte de la definición de tabla externa. Si hay particiones definidas, se anexan al patrón de URI.
Por ejemplo, si la cadena de conexión es `https://storageaccount.blob.core.windows.net/container1` : y hay definida una partición DateTime `partition by format_datetime="yyyy-MM-dd" bin(Timestamp, 1d)`:, el patrón de URI correspondiente sería: `https://storageaccount.blob.core.windows.net/container1/yyyy-MM-dd`y buscaremos blobs en ubicaciones que coincidan con este patrón.
Si hay una partición `"CustomerId" customerId` de cadena adicional definida, el patrón de URI correspondiente es: `https://storageaccount.blob.core.windows.net/container1/yyyy-MM-dd/CustomerId=*`, etc.

2. En el caso de todos los blobs *directos* que se encuentran en los patrones de URI que ha creado, compruebe lo siguiente:

 * Los valores de partición coinciden con predicados usados en una consulta.
 * El nombre del BLOB `NamePrefix`comienza por, si se define una propiedad de este tipo.
 * El nombre del BLOB `FileExtension`termina con, si se define una propiedad de este tipo.

Una vez que se cumplen todas las condiciones, el motor de consultas captura y procesa el BLOB.

#### <a name="spark-virtual-columns-support"></a>Compatibilidad con columnas virtuales de Spark

Cuando los datos se exportan desde Spark, las columnas de partición (que se especifican en el método del escritor de `partitionBy` tramas de datos) no se escriben en los archivos de datos. Esto se hace para evitar la duplicación de datos porque los datos que ya están presentes en los nombres de `column1=<value>/column2=<value>/`"carpeta" (por ejemplo,) y Spark pueden reconocerlos cuando se leen. Sin embargo, Kusto requiere que las columnas de partición estén presentes en los propios datos. La compatibilidad con las columnas virtuales en Kusto está planeada. Hasta entonces, use la siguiente solución alternativa: al exportar datos de Spark, cree una copia de todas las columnas por las que se han particionado los datos antes de escribir una trama de datos:

```kusto
df.withColumn("_a", $"a").withColumn("_b", $"b").write.partitionBy("_a", "_b").parquet("...")
```

Al definir una tabla externa en Kusto, especifique columnas de partición como en el ejemplo siguiente:

```kusto
.create external table ExternalSparkTable(a:string, b:datetime) 
kind=blob
partition by 
   "_a="a,
   format_datetime="'_b='yyyyMMdd" bin(b, 1d)
dataformat=parquet
( 
   h@'https://storageaccount.blob.core.windows.net/container1;secretKey'
)
```

### <a name="show-external-table-artifacts"></a>. Mostrar artefactos de tabla externa

* Devuelve una lista de todos los artefactos que se procesarán al consultar una tabla externa determinada.
* Requiere [permiso de usuario de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis:** 

`.show``external` *TableName* TableName `table``artifacts`

**Salida**

| Parámetro de salida | Tipo   | Descripción                       |
|------------------|--------|-----------------------------------|
| URI              | cadena | URI del artefacto de almacenamiento externo |

**Ejemplos:**

```kusto
.show external table T artifacts
```

**Salida:**

| URI                                                                     |
|-------------------------------------------------------------------------|
| `https://storageaccount.blob.core.windows.net/container1/folder/file.csv` |

### <a name="create-external-table-mapping"></a>. crear asignación de tabla externa

`.create``external` `mapping` *MappingName* *MappingInJsonFormat* *ExternalTableName* ExternalTableName `json` MappingInJsonFormat `table`

Crea una nueva asignación. Para obtener más información, vea [asignaciones de datos](./mappings.md#json-mapping).

**Ejemplo** 
 
```kusto
.create external table MyExternalTable JSON mapping "Mapping1" '[{ "column" : "rownumber", "datatype" : "int", "path" : "$.rownumber"},{ "column" : "rowguid", "path" : "$.rowguid" }]'
```

**Salida del ejemplo**

| NOMBRE     | Clase | Asignación                                                           |
|----------|------|-------------------------------------------------------------------|
| mapping1 | JSON | [{"ColumnName": "RowNumber", "ColumnType": "int", "Properties": {"path": "$. RowNumber"}}, {"ColumnName": "ROWGUID", "ColumnType": "", "propiedades": {"ruta de acceso": "$. ROWGUID"}}] |

### <a name="alter-external-table-mapping"></a>. modificar asignación de tabla externa

`.alter``external` `mapping` *MappingName* *MappingInJsonFormat* *ExternalTableName* ExternalTableName `json` MappingInJsonFormat `table`

Modifica una asignación existente. 
 
**Ejemplo** 
 
```kusto
.alter external table MyExternalTable JSON mapping "Mapping1" '[{ "column" : "rownumber", "path" : "$.rownumber"},{ "column" : "rowguid", "path" : "$.rowguid" }]'
```

**Salida del ejemplo**

| NOMBRE     | Clase | Asignación                                                                |
|----------|------|------------------------------------------------------------------------|
| mapping1 | JSON | [{"ColumnName": "RowNumber", "ColumnType": "", "propiedades": {"ruta de acceso": "$. RowNumber"}}, {"ColumnName": "ROWGUID", "ColumnType": "", "propiedades": {"ruta de acceso": "$. ROWGUID"}}] |

### <a name="show-external-table-mappings"></a>. Mostrar asignaciones de tablas externas

`.show``external` `mapping` *MappingName* *ExternalTableName* ExternalTableName MappingName `table` `json` 

`.show``external` `json` *ExternalTableName* ExternalTableName `table``mappings`

Mostrar las asignaciones (todas o las especificadas por nombre).
 
**Ejemplo** 
 
```kusto
.show external table MyExternalTable JSON mapping "Mapping1" 

.show external table MyExternalTable JSON mappings 
```

**Salida del ejemplo**

| NOMBRE     | Clase | Asignación                                                                         |
|----------|------|---------------------------------------------------------------------------------|
| mapping1 | JSON | [{"ColumnName": "RowNumber", "ColumnType": "", "propiedades": {"ruta de acceso": "$. RowNumber"}}, {"ColumnName": "ROWGUID", "ColumnType": "", "propiedades": {"ruta de acceso": "$. ROWGUID"}}] |

### <a name="drop-external-table-mapping"></a>. quitar asignación de tabla externa

`.drop``external` `mapping` *MappingName* *ExternalTableName* ExternalTableName MappingName `table` `json` 

Quita la asignación de la base de datos.
 
**Ejemplo** 
 
```kusto
.drop external table MyExternalTable JSON mapping "Mapping1" 
```

## <a name="external-sql-table"></a>Tabla SQL externa

### <a name="create-or-alter-external-sql-table"></a>. crear o modificar una tabla SQL externa

Crea o modifica una tabla SQL externa en la base de datos en la que se ejecuta el comando.  

**Sintaxis**

`.create` | (`.alter`) `external` TableName ([ColumnName: columnType],...) *TableName* `table`  
`kind` `=` `sql`  
`table``=` *SqlTableName*  
`(`*SqlServerConnectionString*`)`  
[`with` `(`[`docstring` `=` *FolderName* `=` *value* *Documentation* `folder` `,` *property_name* Documentación] [`,` NombreCarpeta], property_name valor... `=` `)`]


**Parámetros:**

* *TableName* : nombre de tabla externa. Debe seguir las reglas para [los nombres de entidad](../query/schema-entities/entity-names.md). Una tabla externa no puede tener el mismo nombre que una tabla normal de la misma base de datos.
* *SqlTableName* : el nombre de la tabla SQL.
* *SqlServerConnectionString* : la cadena de conexión a SQL Server. Puede ser uno de los siguientes: 
    * **Autenticación integrada** de AAD`Authentication="Active Directory Integrated"`(): el usuario o la aplicación se autentica a través de AAD a Kusto y, a continuación, se usa el mismo token para tener acceso al extremo de red SQL Server.
    * **Autenticación de nombre de usuario/contraseña** (`User ID=...; Password=...;`). Si la tabla externa se utiliza para la [exportación continua](data-export/continuous-data-export.md), la autenticación se debe realizar mediante este método. 

> [!WARNING]
> Las cadenas de conexión y las consultas que incluyen información confidencial se deben ofuscar para que se omitan en cualquier seguimiento de Kusto. Vea [literales de cadena ofuscados](../query/scalar-data-types/string.md#obfuscated-string-literals) para obtener más información.

**Propiedades opcionales**
| Propiedad.            | Tipo            | Descripción                          |
|---------------------|-----------------|---------------------------------------------------------------------------------------------------|
| `folder`            | `string`        | La carpeta de la tabla.                  |
| `docString`         | `string`        | Cadena que documenta la tabla.      |
| `firetriggers`      | `true`/`false`  | Si `true`, indica al sistema de destino que active los desencadenadores de inserción definidos en la tabla SQL. De manera predeterminada, es `false`. (Para obtener más información, vea [Bulk Insert](https://msdn.microsoft.com/library/ms188365.aspx) y [System. Data. SqlClient. SqlBulkCopy).](https://msdn.microsoft.com/library/system.data.sqlclient.sqlbulkcopy(v=vs.110).aspx) |
| `createifnotexists` | `true`/ `false` | Si `true`es, se creará la tabla SQL de destino si aún no existe; en `primarykey` este caso, se debe proporcionar la propiedad para indicar la columna de resultado que es la clave principal. De manera predeterminada, es `false`.  |
| `primarykey`        | `string`        | Si `createifnotexists` es `true`, indica el nombre de la columna en el resultado que se usará como clave principal de la tabla SQL si se crea con este comando.                  |

> [!NOTE]
> * Si la tabla existe, se `.create` producirá un error en el comando. Se `.alter` usa para modificar las tablas existentes. 
> * No se admite la modificación del esquema o el formato de una tabla SQL externa. 

Requiere [permiso de usuario](../management/access-control/role-based-authorization.md) de `.create` base de datos para y `.alter`permiso de administrador de [tablas](../management/access-control/role-based-authorization.md) para. 
 
**Ejemplo** 

```kusto
.create external table ExternalSql (x:long, s:string) 
kind=sql
table=MySqlTable
( 
   h@'Server=tcp:myserver.database.windows.net,1433;Authentication=Active Directory Integrated;Initial Catalog=mydatabase;'
)
with 
(
   docstring = "Docs",
   folder = "ExternalTables", 
   createifnotexists = true,
   primarykey = x,
   firetriggers=true
)  
```

**Salida**

| TableName   | TableType | Carpeta         | DocString | Propiedades                            |
|-------------|-----------|----------------|-----------|---------------------------------------|
| ExternalSql | Sql       | ExternalTables | Docs      | {<br>  "TargetEntityKind": "sqltable",<br>  "TargetEntityName": "MySqlTable",<br>  "TargetEntityConnectionString": "Server = TCP:myserver. Database. Windows. net, 1433; Authentication = Active Directory integrado; Initial Catalog = base de datos; ",<br>  "FireTriggers": true,<br>  "CreateIfNotExists": true,<br>  "PrimaryKey": "x"<br>} |

### <a name="querying-an-external-table-of-type-sql"></a>Consultar una tabla externa de tipo SQL 
De forma similar a otros tipos de tablas externas, se admite la consulta de una tabla SQL externa. Consulte [consultar tablas externas](https://docs.microsoft.com/azure/data-explorer/data-lake-query-data). Tenga en cuenta que la implementación de consultas de tabla externa de SQL ejecutará una ' SELECT * ' completa (o seleccionará columnas relevantes) de la tabla SQL, mientras que el resto de la consulta se ejecutará en el lado de Kusto. Considere la siguiente consulta de tabla externa: 

```kusto
external_table('MySqlExternalTable') | count
```

Kusto ejecutará una consulta ' SELECT * FROM TABLE ' en la base de datos SQL, seguida de un recuento en el lado de Kusto. En tales casos, se espera que el rendimiento sea mejor si se escribe directamente en T-SQL (' SELECT COUNT (1) FROM TABLE ') y se ejecuta con el [complemento sql_request](../query/sqlrequestplugin.md), en lugar de usar la función de tabla externa. Del mismo modo, los filtros no se insertan en la consulta SQL.  
Se recomienda usar la tabla externa para consultar la tabla de SQL cuando la consulta requiera leer toda la tabla (o columnas relevantes) para una mayor ejecución en el lado de Kusto. Cuando una consulta SQL se puede optimizar considerablemente en T-SQL, use el [complemento sql_request](../query/sqlrequestplugin.md).
