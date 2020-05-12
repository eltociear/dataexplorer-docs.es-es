---
title: Tablas externas en Azure Storage o Azure Data Lake-Azure Explorador de datos
description: En este artículo se describe la administración de tablas externas en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: db99d1d46c321bff0f5d7b370766900ea7d1d5a0
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227730"
---
# <a name="external-tables-in-azure-storage-or-azure-data-lake"></a>Tablas externas en Azure Storage o Azure Data Lake

El siguiente comando describe cómo crear una tabla externa. La tabla se puede encontrar en Azure Blob Storage, Azure Data Lake Store GEN1 o Azure Data Lake Store de la segunda generación. 
[Cadenas de conexión de almacenamiento](../api/connection-strings/storage.md) describe la creación de la cadena de conexión para cada una de estas opciones. 

## <a name="create-or-alter-external-table"></a>. Create o. Alter external Table

**Sintaxis**

( `.create`  |  `.alter` ) `external` `table` *TableName* (*esquema*)  
`kind` `=` (`blob` | `adl`)  
[ `partition` `by` *Partition* [ `,` ....]]  
`dataformat` `=` *Formato*  
`(`  
*StorageConnectionString* [ `,` ...]  
`)`  
[ `with` `(` [ `docstring` `=` *Documentación*] [ `,` `folder` `=` *nombrecarpeta*], *property_name* `=` *valor* `,` ... `)` ]

Crea o modifica una nueva tabla externa en la base de datos en la que se ejecuta el comando.

**Parámetros**

* *TableName* : nombre de tabla externa. Debe seguir las reglas para [los nombres de entidad](../query/schema-entities/entity-names.md). Una tabla externa no puede tener el mismo nombre que una tabla normal de la misma base de datos.
* Esquema *-esquema de datos* externos en formato: `ColumnName:ColumnType[, ColumnName:ColumnType ...]` . Si no se conoce el esquema de datos externos, use el complemento de [infer_storage_schema](../query/inferstorageschemaplugin.md) , que puede deducir el esquema en función del contenido del archivo externo.
* *Partición* : una o varias definiciones de partición (opcional). Vea la sintaxis de partición a continuación.
* *Format* : formato de datos. Se admite cualquiera de los [formatos de ingesta](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats) para realizar consultas. El uso de una tabla externa para el [escenario de exportación](data-export/export-data-to-an-external-table.md) se limita a los siguientes formatos: `CSV` , `TSV` , `JSON` , `Parquet` .
* *StorageConnectionString* : una o varias rutas de acceso para Azure BLOB Storage contenedores de blobs o sistemas de archivos Azure Data Lake Store (directorios virtuales o carpetas), incluidas las credenciales. Consulte [cadenas de conexión de almacenamiento](../api/connection-strings/storage.md) para obtener más información. Proporcione más que una sola cuenta de almacenamiento para evitar la limitación de almacenamiento al [exportar](data-export/export-data-to-an-external-table.md) grandes cantidades de datos a la tabla externa. La exportación distribuirá las escrituras entre todas las cuentas proporcionadas. 

**Sintaxis de partición**

[ `format_datetime =` *DateTimePartitionFormat*] `bin(` *TimestampColumnName*, *PartitionByTimeSpan*`)`  
|   
[*StringFormatPrefix*] *StringColumnName* [*StringFormatSuffix*])

**Parámetros de partición**

* *DateTimePartitionFormat* : el formato de la estructura de directorios necesaria en la ruta de acceso de salida (opcional). Si se define la creación de particiones y el formato no se especifica, el valor predeterminado es "AAAA/MM/DD/HH/mm". Este formato se basa en PartitionByTimeSpan. Por ejemplo, si crea una partición por 1D, Structure será "AAAA/MM/DD". Si crea particiones en 1 h, la estructura será "AAAA/MM/DD/HH".
* *TimestampColumnName* : columna de fecha y hora en la que se crean las particiones de la tabla. La columna de marca de tiempo debe existir en la definición de esquema de tabla externa y en la salida de la consulta de exportación al exportar a la tabla externa.
* *PartitionByTimeSpan* : literal de TimeSpan por el que se va a particionar.
* *StringFormatPrefix* : un literal de cadena constante que formará parte de la ruta de acceso del artefacto, concatenado antes del valor de la tabla (opcional).
* *StringFormatSuffix* : un literal de cadena constante que formará parte de la ruta de acceso del artefacto, concatenado después del valor de la tabla (opcional).
* *StringColumnName* : columna de cadena en la que se crean las particiones de la tabla. La columna de cadena debe existir en la definición de esquema de tabla externa.

**Propiedades opcionales**:

| Propiedad         | Tipo     | Descripción       |
|------------------|----------|-------------------------------------------------------------------------------------|
| `folder`         | `string` | Carpeta de la tabla                                                                     |
| `docString`      | `string` | Cadena que documenta la tabla                                                       |
| `compressed`     | `bool`   | Si se establece, indica si los blobs se comprimen como `.gz` archivos                  |
| `includeHeaders` | `string` | En el caso de los blobs CSV o TSV, indica si los blobs contienen un encabezado                     |
| `namePrefix`     | `string` | Si se establece, indica el prefijo de los BLOBs. En las operaciones de escritura, todos los blobs se escribirán con este prefijo. En las operaciones de lectura, solo se leen los blobs con este prefijo. |
| `fileExtension`  | `string` | Si se establece, indica extensiones de archivo de los BLOBs. Al escribir, los nombres de los blobs terminarán con este sufijo. En la lectura, solo se leerán los blobs con esta extensión de archivo.           |
| `encoding`       | `string` | Indica cómo se codifica el texto: `UTF8NoBOM` (valor predeterminado) o `UTF8BOM` .             |

Para obtener más información sobre los parámetros de tabla externa en las consultas, vea [lógica de filtrado de artefactos](#artifact-filtering-logic).

> [!NOTE]
> * Si la tabla existe, `.create` se producirá un error en el comando. `.alter`Se usa para modificar las tablas existentes. 
> * No se admite la modificación del esquema, el formato o la definición de partición de una tabla de blobs externos. 

Requiere [permiso de usuario de base de datos](../management/access-control/role-based-authorization.md) para `.create` y [permiso de administrador de tablas](../management/access-control/role-based-authorization.md) para `.alter` . 
 
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

Una tabla externa con particiones por fecha y hora. Los artefactos están en los directorios con el formato "AAAA/MM/DD" en las rutas de acceso definidas:

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
|ExternalMultiplePartitions|Blob|ExternalTables|Docs|{"Format": "CSV", "Compressed": false, "CompressionType": null, "FileExtension": "CSV", "IncludeHeaders": "none", "Encoding": null, "NamePrefix": null}|["https://storageaccount.blob.core.windows.net/container1;*******"]}|[{"StringFormat": "CustomerName = {0} ", "ColumnName": "customername", "ordinal": 0}, PartitionBy ":" 1.00:00:00 "," ColumnName ":" timestamp "," ordinal ": 1}]|

### <a name="artifact-filtering-logic"></a>Lógica de filtrado de artefactos

Al consultar una tabla externa, el motor de consultas mejora el rendimiento al filtrar artefactos de almacenamiento externo irrelevantes (blobs). A continuación se describe el proceso de iteración en blobs y la determinación de si se debe procesar un BLOB.

1. Cree un patrón de URI que represente un lugar donde se encuentren los BLOBs. Inicialmente, el patrón de URI es igual a una cadena de conexión proporcionada como parte de la definición de tabla externa. Si hay particiones definidas, se anexan al patrón de URI.
Por ejemplo, si la cadena de conexión es: `https://storageaccount.blob.core.windows.net/container1` y hay definida una partición DateTime: `partition by format_datetime="yyyy-MM-dd" bin(Timestamp, 1d)` , el patrón de URI correspondiente sería: y `https://storageaccount.blob.core.windows.net/container1/yyyy-MM-dd` buscaremos blobs en ubicaciones que coincidan con este patrón.
Si hay una partición de cadena adicional `"CustomerId" customerId` definida, el patrón de URI correspondiente es: `https://storageaccount.blob.core.windows.net/container1/yyyy-MM-dd/CustomerId=*` .

1. En el caso de todos los blobs *directos* que se encuentran en los patrones de URI que ha creado, compruebe lo siguiente:

   * Los valores de partición coinciden con predicados usados en una consulta.
   * El nombre del BLOB comienza por `NamePrefix` , si se define una propiedad de este tipo.
   * El nombre del BLOB termina con `FileExtension` , si se define una propiedad de este tipo.

Una vez que se cumplen todas las condiciones, el motor de consultas captura y procesa el BLOB.

### <a name="spark-virtual-columns-support"></a>Compatibilidad con columnas virtuales de Spark

Cuando los datos se exportan desde Spark, las columnas de partición (que se especifican en el método del escritor de tramas de datos `partitionBy` ) no se escriben en los archivos de datos. Este proceso evita la duplicación de datos porque los datos ya están presentes en nombres de "carpeta". Por ejemplo, `column1=<value>/column2=<value>/` y Spark puede reconocerlo en el momento de la lectura. Sin embargo, Kusto requiere que las columnas de partición estén presentes en los propios datos. La compatibilidad con las columnas virtuales en Kusto está planeada. Hasta entonces, use la siguiente solución alternativa: al exportar datos de Spark, cree una copia de todas las columnas por las que se han particionado los datos antes de escribir una trama de datos:

```kusto
df.withColumn("_a", $"a").withColumn("_b", $"b").write.partitionBy("_a", "_b").parquet("...")
```

Al definir una tabla externa en Kusto, especifique las columnas de partición como en el ejemplo siguiente:

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

## <a name="show-external-table-artifacts"></a>. Mostrar artefactos de tabla externa

* Devuelve una lista de todos los artefactos que se procesarán al consultar una tabla externa determinada.
* Requiere [permiso de usuario de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis:** 

`.show``external` `table` *TableName*`artifacts`

**Salida**

| Parámetro de salida | Tipo   | Descripción                       |
|------------------|--------|-----------------------------------|
| URI              | string | URI del artefacto de almacenamiento externo |

**Ejemplos:**

```kusto
.show external table T artifacts
```

**Salida:**

| URI                                                                     |
|-------------------------------------------------------------------------|
| `https://storageaccount.blob.core.windows.net/container1/folder/file.csv` |

## <a name="create-external-table-mapping"></a>. crear asignación de tabla externa

`.create``external` `table` *ExternalTableName* `json` `mapping` *MappingName* *MappingInJsonFormat*

Crea una nueva asignación. Para obtener más información, vea [asignaciones de datos](./mappings.md#json-mapping).

**Ejemplo** 
 
```kusto
.create external table MyExternalTable JSON mapping "Mapping1" '[{ "column" : "rownumber", "datatype" : "int", "path" : "$.rownumber"},{ "column" : "rowguid", "path" : "$.rowguid" }]'
```

**Salida del ejemplo**

| Nombre     | Clase | Asignación                                                           |
|----------|------|-------------------------------------------------------------------|
| mapping1 | JSON | [{"ColumnName": "RowNumber", "ColumnType": "int", "Properties": {"path": "$. RowNumber"}}, {"ColumnName": "ROWGUID", "ColumnType": "", "propiedades": {"ruta de acceso": "$. ROWGUID"}}] |

## <a name="alter-external-table-mapping"></a>. modificar asignación de tabla externa

`.alter``external` `table` *ExternalTableName* `json` `mapping` *MappingName* *MappingInJsonFormat*

Modifica una asignación existente. 
 
**Ejemplo** 
 
```kusto
.alter external table MyExternalTable JSON mapping "Mapping1" '[{ "column" : "rownumber", "path" : "$.rownumber"},{ "column" : "rowguid", "path" : "$.rowguid" }]'
```

**Salida del ejemplo**

| Nombre     | Clase | Asignación                                                                |
|----------|------|------------------------------------------------------------------------|
| mapping1 | JSON | [{"ColumnName": "RowNumber", "ColumnType": "", "propiedades": {"ruta de acceso": "$. RowNumber"}}, {"ColumnName": "ROWGUID", "ColumnType": "", "propiedades": {"ruta de acceso": "$. ROWGUID"}}] |

## <a name="show-external-table-mappings"></a>. Mostrar asignaciones de tablas externas

`.show``external` `table` *ExternalTableName* `json` `mapping` *MappingName* 

`.show``external` `table` *ExternalTableName* ExternalTableName `json``mappings`

Mostrar las asignaciones (todas o las especificadas por nombre).
 
**Ejemplo** 
 
```kusto
.show external table MyExternalTable JSON mapping "Mapping1" 

.show external table MyExternalTable JSON mappings 
```

**Salida del ejemplo**

| Nombre     | Clase | Asignación                                                                         |
|----------|------|---------------------------------------------------------------------------------|
| mapping1 | JSON | [{"ColumnName": "RowNumber", "ColumnType": "", "propiedades": {"ruta de acceso": "$. RowNumber"}}, {"ColumnName": "ROWGUID", "ColumnType": "", "propiedades": {"ruta de acceso": "$. ROWGUID"}}] |

## <a name="drop-external-table-mapping"></a>. quitar asignación de tabla externa

`.drop``external` `table` *ExternalTableName* `json` `mapping` *MappingName* 

Quita la asignación de la base de datos.
 
**Ejemplo** 
 
```kusto
.drop external table MyExternalTable JSON mapping "Mapping1" 
```
