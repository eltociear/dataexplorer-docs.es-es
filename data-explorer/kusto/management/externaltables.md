---
title: 'Administración de tablas externas: Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe Administración de tablas externas en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 680a4e25d6b478fe171aa3296de81c0106417877
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744714"
---
# <a name="external-table-management"></a>Gestión externa de tablas

Consulte [tablas externas](../query/schema-entities/externaltables.md) para obtener información general sobre las tablas externas. 

## <a name="common-external-tables-control-commands"></a>Comandos comunes de control de tablas externas

Los siguientes comandos son relevantes para _cualquier_ tabla externa (de cualquier tipo).

### <a name="show-external-tables"></a>.show tablas externas

* Devuelve todas las tablas externas de la base de datos (o una tabla externa específica).
* Requiere [permiso](../management/access-control/role-based-authorization.md)de supervisión de base de datos .

**Sintaxis:** 

`.show` `external` `tables`

`.show``external` `table` *TableName*

**Salida**

| Parámetro de salida | Tipo   | Descripción                                                         |
|------------------|--------|---------------------------------------------------------------------|
| TableName        | string | Nombre de la tabla externa                                             |
| TableType        | string | Tipo de tabla externa                                              |
| Carpeta           | string | Carpeta de la tabla                                                     |
| DocString        | string | Cadena que documenta la tabla                                       |
| Propiedades       | string | Propiedades serializadas JSON de la tabla (específicas para el tipo de tabla) |


**Ejemplos:**

```kusto
.show external tables
.show external table T
```

| TableName | TableType | Carpeta         | DocString | Propiedades |
|-----------|-----------|----------------|-----------|------------|
| T         | Blob      | ExternalTables | Docs      | {}         |


### <a name="show-external-table-schema"></a>Esquema de tabla externa .show

* Devuelve el esquema de la tabla externa, como JSON o CSL. 
* Requiere [permiso](../management/access-control/role-based-authorization.md)de supervisión de base de datos .

**Sintaxis:** 

`.show``external` `as` *TableName* `json`TableName | (`csl`) `table` `schema`

`.show``external` `table` *TableName*`cslschema`

**Salida**

| Parámetro de salida | Tipo   | Descripción                        |
|------------------|--------|------------------------------------|
| TableName        | string | Nombre de la tabla externa            |
| Schema           | string | El esquema de tabla en formato JSON |
| DatabaseName     | string | Nombre de la base de datos de la tabla             |
| Carpeta           | string | Carpeta de la tabla                    |
| DocString        | string | Cadena que documenta la tabla      |

**Ejemplos:**

```kusto
.show external table T schema as JSON
```

```kusto
.show external table T schema as CSL
.show external table T cslschema
```

**Salida:**

*Json:*

| TableName | Schema    | DatabaseName | Carpeta         | DocString |
|-----------|----------------------------------|--------------|----------------|-----------|
| T         | "Name":"ExternalBlob",<br>"Folder":"ExternalTables",<br>"DocString":"Docs",<br>"OrderedColumns":['"Name":"x","Type":"System.Int64","CslType":"long","DocString":""","Name":"s","Type":"System.String","CslType":"string","DocString":"""-]. | DB           | ExternalTables | Docs      |


*Csl:*

| TableName | Schema          | DatabaseName | Carpeta         | DocString |
|-----------|-----------------|--------------|----------------|-----------|
| T         | x:long,s:string | DB           | ExternalTables | Docs      |

### <a name="drop-external-table"></a>Tabla externa .drop

* Suelta una tabla externa 
* La definición de tabla externa no se puede restaurar después de esta operación
* Requiere [permiso](../management/access-control/role-based-authorization.md)de administrador de base de datos .

**Sintaxis:**  

`.drop``external` `table` *TableName*

**Salida**

Devuelve las propiedades de la tabla descartada. Consulte [Tablas externas .show](#show-external-tables).

**Ejemplos:**

```kusto
.drop external table ExternalBlob
```

| TableName | TableType | Carpeta         | DocString | Schema       | Propiedades |
|-----------|-----------|----------------|-----------|-----------------------------------------------------|------------|
| T         | Blob      | ExternalTables | Docs      | ['Nombre': "x", "CslType": "long"-,<br> • "Nombre": "s", "CslType": "string" ] | {}         |

## <a name="external-tables-in-azure-storage-or-azure-data-lake"></a>Tablas externas en Azure Storage o Azure Data Lake

El siguiente comando describe cómo crear una tabla externa. La tabla se puede encontrar en Azure Blob Storage, Azure Data Lake Store Gen1 o Azure Data Lake Store Gen2. 
[Las cadenas](../api/connection-strings/storage.md) de conexión de almacenamiento describen la creación de la cadena de conexión para cada una de estas opciones. 

### <a name="create-or-alter-external-table"></a>Tabla externa .create o .alter

**Sintaxis**

`.create` | (`.alter` `external` ) `table` *TableName* (*Esquema*)  
`kind` `=` (`blob` | `adl`)  
[`partition` `by` *Partición* [`,` ....]]  
`dataformat` `=` *Formato*  
`(`  
*StorageConnectionString* `,` [ ...]  
`)`  
[`with` `(``docstring` `=` [ *Documentación*`,` `folder` `=` ] [ *NombreCarpeta*], *property_name* `=` *valor*`,`... `)`]

Crea o modifica una nueva tabla externa en la base de datos en la que se ejecuta el comando.

**Parámetros**

* *TableName* - Nombre de tabla externa. Debe seguir las reglas para los nombres de [entidad](../query/schema-entities/entity-names.md). Una tabla externa no puede tener el mismo nombre que una tabla normal en la misma base de datos.
* *Esquema* - Esquema de `ColumnName:ColumnType[, ColumnName:ColumnType ...]`datos externos en formato: . Si el esquema de datos externos es desconocido, utilice el complemento [infer_storage_schema,](../query/inferstorageschemaplugin.md) que puede deducir el esquema en función del contenido del archivo externo.
* *Partición:* una o varias definiciones de partición (opcional). Consulte la sintaxis de partición a continuación.
* *Formato* - El formato de datos. Cualquiera de los formatos de [ingesta](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats) se admite para realizar consultas. El uso de la tabla externa para el `JSON` `Parquet`escenario de [exportación](data-export/export-data-to-an-external-table.md) se limita a los siguientes formatos: `CSV`, `TSV`, , .
* *StorageConnectionString:* una o varias rutas de acceso a contenedores de blobs de Azure Blob Storage o sistemas de archivos de Azure Data Lake Store (directorios o carpetas virtuales), incluidas las credenciales. Consulte Cadenas de conexión de [almacenamiento](../api/connection-strings/storage.md) para obtener más información. Se recomienda encarecidamente proporcionar más de una sola cuenta de almacenamiento para evitar la limitación del almacenamiento si [exporta](data-export/export-data-to-an-external-table.md) grandes cantidades de datos a la tabla externa. La exportación distribuirá las escrituras entre todas las cuentas proporcionadas. 

**Sintaxis de partición**

[`format_datetime =` *DateTimePartitionFormat*] `bin(` *TimestampColumnName*, *PartitionByTimeSpan*`)`  
|   
[*StringFormatPrefix*] *StringColumnName* [*StringFormatSuffix*])

**Parámetros de partición**

* *DateTimePartitionFormat:* el formato de la estructura de directorios necesaria en la ruta de acceso de salida (opcional). Si se define la partición y no se especifica el formato, el valor predeterminado es "aaaa/MM/dd/HH/mm", basado en PartitionByTimeSpan. Por ejemplo, si particiona por 1d, la estructura será "aaaa/MM/dd". Si particiona por 1h, la estructura será "aaaa/MM/dd/HH".
* *TimestampColumnName* - columna Datetime en la que se particiona la tabla. La columna de marca de tiempo debe existir en la definición y salida del esquema de tabla externa de la consulta de exportación, al exportar a la tabla externa.
* *PartitionByTimeSpan* - Literal de timespan por el que se va a particionar.
* *StringFormatPrefix* : literal de cadena constante que formará parte de la ruta de acceso del artefacto, concatenado antes del valor de la tabla (opcional).
* *StringFormatSuffix:* literal de cadena constante que formará parte de la ruta de acceso del artefacto, concatenado después del valor de la tabla (opcional).
* *StringColumnName* : columna de cadena en la que se particiona la tabla. La columna de cadena debe existir en la definición de esquema de tabla externa.

**Propiedades opcionales**:

| Propiedad         | Tipo     | Descripción       |
|------------------|----------|-------------------------------------------------------------------------------------|
| `folder`         | `string` | Carpeta de la tabla                                                                     |
| `docString`      | `string` | Cadena que documenta la tabla                                                       |
| `compressed`     | `bool`   | Si se establece, indica si los `.gz` blobs se comprimen como archivos                  |
| `includeHeaders` | `string` | Para blobs CSV o TSV, indica si los blobs contienen un encabezado                     |
| `namePrefix`     | `string` | Si se establece, indica el prefijo de los blobs. En las operaciones de escritura, todos los blobs se escribirán con este prefijo. En las operaciones de lectura, solo se leen los blobs con este prefijo. |
| `fileExtension`  | `string` | Si se establece, indica las extensiones de archivo de los blobs. Al escribir, los nombres de blobs terminarán con este sufijo. En lectura, solo se leerán los blobs con esta extensión de archivo.           |
| `encoding`       | `string` | Indica cómo se codifica `UTF8NoBOM` el texto: `UTF8BOM`(predeterminado) o .             |

> [!NOTE]
> * Si la tabla `.create` existe, el comando fallará con un error. Se `.alter` utiliza para modificar las tablas existentes. 
> * No se admite la modificación del esquema, el formato o la definición de partición de una tabla de blobs externa. 

Requiere [permiso](../management/access-control/role-based-authorization.md) de `.create` usuario de base `.alter`de datos y permiso de administrador de [tabla](../management/access-control/role-based-authorization.md) para . 
 
**Ejemplo** 

Una tabla externa no particionada. Se espera que todos los artefactos estén directamente debajo de los contenedores definidos:

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
   folder = "ExternalTables",
   namePrefix="Prefix"
)  
```

Una tabla externa particionada por dateTime. Los artefactos residen en directorios en formato "aaaa/MM/dd" bajo las rutas definidas:

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
   folder = "ExternalTables",
   namePrefix="Prefix"
)  
```

Una tabla externa particionada por dateTime con un formato de directorio de "year-yyyy/month-MM/day-dd":

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
   folder = "ExternalTables",
   namePrefix="Prefix"
)
```

Una tabla externa con particiones de datos mensuales y un formato de directorio de "aaaa/MM":

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
   folder = "ExternalTables",
   namePrefix="Prefix"
)
```

Una tabla externa con dos particiones. La estructura de directorios es la concatenación de ambas particiones: con formato CustomerName seguido del formato dateTime predeterminado. Por ejemplo, "NombreDeCliente/Softworks/2011/11/11":

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
|ExternalMultiplePartitions|Blob|ExternalTables|Docs|"Format":"Csv","Compressed":false,"CompressionType":null,"FileExtension":"csv","IncludeHeaders":"None","Encoding":null,"NamePrefix":null"|["https://storageaccount.blob.core.windows.net/container1;*******"]}|['"StringFormat":"NombreDeCliente-"NombreDeColumna":"NombreDeCliente","Ordinal":0-,PartitionBy":"1.00:00:00","ColumnName":"Timestamp","Ordinal":1-]{0}|

#### <a name="spark-virtual-columns-support"></a>Compatibilidad con columnas virtuales de Spark

Cuando se exportan datos desde Spark, las columnas de `partitionBy` partición (que se especifican en el método del escritor de marcos de datos) no se escriben en los archivos de datos. Esto se hace para evitar la duplicación de datos porque `column1=<value>/column2=<value>/`los datos ya están presentes en los nombres de "carpeta" (por ejemplo, ), y Spark puede reconocerlo al leer. Sin embargo, Kusto requiere que las columnas de partición estén presentes en los propios datos. Se planea la compatibilidad con columnas virtuales en Kusto. Hasta entonces, utilice la siguiente solución alternativa: al exportar datos desde Spark, cree una copia de todas las columnas por las que se particionen los datos antes de escribir una trama de datos:

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

### <a name="show-external-table-artifacts"></a>.show artefactos de tabla externa

* Devuelve una lista de todos los artefactos que se procesarán al consultar una tabla externa determinada.
* Requiere [permiso](../management/access-control/role-based-authorization.md)de usuario de base de datos .

**Sintaxis:** 

`.show``external` `table` *TableName*`artifacts`

**Salida**

| Parámetro de salida | Tipo   | Descripción                       |
|------------------|--------|-----------------------------------|
| Identificador URI              | string | URI del artefacto de almacenamiento externo |

**Ejemplos:**

```kusto
.show external table T artifacts
```

**Salida:**

| Identificador URI                                                                     |
|-------------------------------------------------------------------------|
| https://storageaccount.blob.core.windows.net/container1/folder/file.csv |

### <a name="create-external-table-mapping"></a>Asignación de tablaexterna .create

`.create``external` `mapping` *MappingName* *ExternalTableName* `json` *MappingInJsonFormat* ExternalTableName MappingName MappingInJsonFormat `table`

Crea una nueva asignación. Para obtener más información, consulte [Asignaciones](./mappings.md#json-mapping)de datos .

**Ejemplo** 
 
```kusto
.create external table MyExternalTable JSON mapping "Mapping1" '[{ "column" : "rownumber", "datatype" : "int", "path" : "$.rownumber"},{ "column" : "rowguid", "path" : "$.rowguid" }]'
```

**Salida del ejemplo**

| NOMBRE     | Clase | Asignación                                                           |
|----------|------|-------------------------------------------------------------------|
| mapeo1 | JSON | ['"ColumnName":"rownumber","ColumnType":"int","Properties":'"Path":"$.rownumber"','ColumnName":"rowguid","ColumnType":""Properties":"'''Path":"$.rowguid'']. |

### <a name="alter-external-table-mapping"></a>Asignación de tablas externas .alter

`.alter``external` `mapping` *MappingName* *ExternalTableName* `json` *MappingInJsonFormat* ExternalTableName MappingName MappingInJsonFormat `table`

Altera una asignación existente. 
 
**Ejemplo** 
 
```kusto
.alter external table MyExternalTable JSON mapping "Mapping1" '[{ "column" : "rownumber", "path" : "$.rownumber"},{ "column" : "rowguid", "path" : "$.rowguid" }]'
```

**Salida del ejemplo**

| NOMBRE     | Clase | Asignación                                                                |
|----------|------|------------------------------------------------------------------------|
| mapeo1 | JSON | ['"ColumnName":"rownumber","ColumnType":"","Properties":'Path":"$.rownumber" '','ColumnName":"rowguid","ColumnType":""'Properties':''Path":"$.rowguid'']. |

### <a name="show-external-table-mappings"></a>.show asignaciones de tablas externas

`.show``external` `mapping` *ExternalTableName* `json` ExternalTableName *MappingName* `table` 

`.show``external` *ExternalTableName* `json` ExternalTableName `table``mappings`

Mostrar las asignaciones (todas o la especificada por nombre).
 
**Ejemplo** 
 
```kusto
.show external table MyExternalTable JSON mapping "Mapping1" 

.show external table MyExternalTable JSON mappings 
```

**Salida del ejemplo**

| NOMBRE     | Clase | Asignación                                                                         |
|----------|------|---------------------------------------------------------------------------------|
| mapeo1 | JSON | ['"ColumnName":"rownumber","ColumnType":"","Properties":'Path":"$.rownumber" '','ColumnName":"rowguid","ColumnType":""'Properties':''Path":"$.rowguid'']. |

### <a name="drop-external-table-mapping"></a>Asignación de tabla externa .drop

`.drop``external` `mapping` *ExternalTableName* `json` ExternalTableName *MappingName* `table` 

Quita la asignación de la base de datos.
 
**Ejemplo** 
 
```kusto
.drop external table MyExternalTable JSON mapping "Mapping1" 
```

## <a name="external-sql-table"></a>Tabla SQL externa

### <a name="create-or-alter-external-sql-table"></a>.create o alterar la tabla sql externa

Crea o modifica una tabla SQL externa en la base de datos en la que se ejecuta el comando.  

**Sintaxis**

`.create` | (`.alter` `external` ) `table` *TableName* ([columnName:columnType], ...)  
`kind` `=` `sql`  
`table``=` *SqlTableName*  
`(`*SqlServerConnectionString*`)`  
[`with` `(``docstring` `=` [ *Documentación*`,` `folder` `=` ] [ *NombreCarpeta*], *property_name* `=` *valor*`,`... `)`]


**Parámetros:**

* *TableName* - Nombre de tabla externa. Debe seguir las reglas para los nombres de [entidad](../query/schema-entities/entity-names.md). Una tabla externa no puede tener el mismo nombre que una tabla normal en la misma base de datos.
* *SqlTableName:* el nombre de la tabla SQL.
* *SqlServerConnectionString:* la cadena de conexión al servidor SQL. Puede ser uno de los siguientes: 
    * **Autenticación integrada** de`Authentication="Active Directory Integrated"`AAD ( ): el usuario o la aplicación se autentica a través de AAD en Kusto y, a continuación, se usa el mismo token para tener acceso al extremo de red de SQL ServerSQL Server .
    * Autenticación de nombre`User ID=...; Password=...;`de **usuario/contraseña** ( ). Si la tabla externa se utiliza para la [exportación continua,](data-export/continuous-data-export.md)la autenticación debe realizarse mediante este método. 

> [!WARNING]
> Las cadenas de conexión y las consultas que incluyen información confidencial deben ofuscarse para que se omitan de cualquier seguimiento de Kusto. Consulte literales de [cadena ofuscados](../query/scalar-data-types/string.md#obfuscated-string-literals) para obtener más información.

**Propiedades opcionales**
| Propiedad            | Tipo            | Descripción                          |
|---------------------|-----------------|---------------------------------------------------------------------------------------------------|
| `folder`            | `string`        | La carpeta de la tabla.                  |
| `docString`         | `string`        | Una cadena que documenta la tabla.      |
| `firetriggers`      | `true`/`false`  | Si `true`, indica al sistema de destino que active los desencadenadores INSERT definidos en la tabla SQL. De manera predeterminada, es `false`. (Para obtener más información, vea [BULK INSERT](https://msdn.microsoft.com/library/ms188365.aspx) y [System.Data.SqlClient.SqlBulkCopy](https://msdn.microsoft.com/library/system.data.sqlclient.sqlbulkcopy(v=vs.110).aspx)) |
| `createifnotexists` | `true`/ `false` | Si `true`, se creará la tabla SQL de destino si aún no existe; la `primarykey` propiedad debe proporcionarse en este caso para indicar la columna de resultados que es la clave principal. De manera predeterminada, es `false`.  |
| `primarykey`        | `string`        | Si `createifnotexists` `true`es , indica el nombre de la columna en el resultado que se usará como clave principal de la tabla SQL si se crea mediante este comando.                  |

> [!NOTE]
> * Si la tabla existe, el `.create` comando producirá un error. Se `.alter` utiliza para modificar las tablas existentes. 
> * No se admite la modificación del esquema o el formato de una tabla SQL externa. 

Requiere [permiso](../management/access-control/role-based-authorization.md) de `.create` usuario de base `.alter`de datos y permiso de administrador de [tabla](../management/access-control/role-based-authorization.md) para . 
 
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
| ExternalSql | Sql       | ExternalTables | Docs      | {<br>  "TargetEntityKind": "sqltable",<br>  "TargetEntityName": "MySqlTable",<br>  "TargetEntityConnectionString": "Servidor-tcp:myserver.database.windows.net,1433; Autenticación-Active Directory Integrado;Catálogo Inicial-mibase de datos;",<br>  "FireTriggers": true,<br>  "CreateIfNotExists": true,<br>  "PrimaryKey": "x"<br>} |

### <a name="querying-an-external-table-of-type-sql"></a>Consultar una tabla externa de tipo SQL 
Similar a otros tipos de tablas externas, se admite la consulta de una tabla SQL externa. Consulte [consulta de tablas externas](https://docs.microsoft.com/azure/data-explorer/data-lake-query-data). Tenga en cuenta que la implementación de la consulta de tabla externa DE SQL ejecutará un 'SELECT *' completo (o seleccionar columnas relevantes) de la tabla SQL, mientras que el resto de la consulta se ejecutará en el lado de Kusto. Considere la siguiente consulta de tabla externa: 

```kusto
external_table('MySqlExternalTable') | count
```

Kusto ejecutará una consulta 'SELECT * from TABLE' a la base de datos SQL, seguida de un recuento en el lado de Kusto. En tales casos, se espera que el rendimiento sea mejor si se escribe en T-SQL directamente ('SELECT COUNT(1) FROM TABLE') y se ejecuta utilizando el [sql_request plugin,](../query/sqlrequestplugin.md)en lugar de utilizar la función de tabla externa. De forma similar, los filtros no se insertan en la consulta SQL.  
Se recomienda utilizar la tabla externa para consultar la tabla SQL cuando la consulta requiera la lectura de toda la tabla (o columnas relevantes) para su posterior ejecución en el lado de Kusto. Cuando una consulta SQL se puede optimizar significativamente en T-SQL, utilice el [complemento sql_request.](../query/sqlrequestplugin.md)
