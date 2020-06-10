---
title: Crear y modificar tablas externas en Azure Storage o Azure Data Lake-Azure Explorador de datos
description: En este artículo se describe cómo crear y modificar tablas externas en Azure Storage o Azure Data Lake
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 296c6e245b7157c09c7af59132fd8bfa686fc9f7
ms.sourcegitcommit: be1bbd62040ef83c08e800215443ffee21cb4219
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2020
ms.locfileid: "84665051"
---
# <a name="create-and-alter-external-tables-in-azure-storage-or-azure-data-lake"></a>Creación y modificación de tablas externas en Azure Storage o Azure Data Lake

El siguiente comando describe cómo crear una tabla externa ubicada en Azure Blob Storage, Azure Data Lake Store GEN1 o Azure Data Lake Store de la generación de código. 

## <a name="create-or-alter-external-table"></a>. Create o. Alter external Table

**Sintaxis**

( `.create`  |  `.alter` ) `external` `table` *[TableName](#table-name)* `(` *[Esquema](#schema)* TableName`)`  
`kind` `=` (`blob` | `adl`)  
[ `partition` `by` `(` *[Particiones](#partitions)* `)` [ `pathformat` `=` `(` *[PathFormat](#path-format)* `)` ]]  
`dataformat``=` * [Formato](#format)*  
`(`*[StorageConnectionString](#connection-string)* [ `,` ...]`)`   
[ `with` `(` Valor de *[PropertyName](#properties)* `=` *[Value](#properties)* `,` ... `)` ]  

Crea o modifica una nueva tabla externa en la base de datos en la que se ejecuta el comando.

> [!NOTE]
> * Si la tabla existe, `.create` se producirá un error en el comando. `.alter`Se usa para modificar las tablas existentes. 
> * No se admite la modificación del esquema, el formato o la definición de partición de una tabla de blobs externos. 
> * La operación requiere el permiso de [usuario de base de datos](../management/access-control/role-based-authorization.md) para y el `.create` [permiso de administrador de tablas](../management/access-control/role-based-authorization.md) para `.alter` . 

**Parámetros**

<a name="table-name"></a>
*NombreTabla*

Nombre de la tabla externa que cumple las reglas de [nombres de entidad](../query/schema-entities/entity-names.md) .
Una tabla externa no puede tener el mismo nombre que una tabla normal de la misma base de datos.

<a name="schema"></a>
*Esquema*

El esquema de datos externos se describe con el siguiente formato:

&nbsp;&nbsp;*ColumnName* `:` *ColumnType* [ `,` *columnName* `:` *ColumnType* ...]

donde *columnName* cumple las reglas de [nomenclatura de entidades](../query/schema-entities/entity-names.md) y *ColumnType* es uno de los tipos de [datos admitidos](../query/scalar-data-types/index.md).

> [!TIP]
> Si no se conoce el esquema de datos externos, use el complemento de [ \_ \_ esquema de almacenamiento de inferencia](../query/inferstorageschemaplugin.md) , que ayuda a deducir el esquema basado en el contenido del archivo externo.

<a name="partitions"></a>
*Particiones*

Lista separada por comas de las columnas por las que se crea una partición de una tabla externa. La columna de partición puede existir en el propio archivo de datos o en la parte SA de la ruta de acceso del archivo (Lea más en [columnas virtuales](#virtual-columns)).

La lista de particiones es cualquier combinación de columnas de partición que se especifica mediante uno de los siguientes formatos:

* Partición que representa una [columna virtual](#virtual-columns).

  *PartitionName* `:` (`datetime` | `string`)

* Partición, basada en un valor de columna de cadena.

  *PartitionName* `:` `string` `=` *ColumnName*

* Partición, basada en un [hash](../query/hashfunction.md)de valor de columna de cadena, *número*de módulo.

  *PartitionName* `:` `long` `=` `hash` `(` *ColumnName* `,` *Número* ColumnName`)`

* Partición, basada en el valor truncado de una columna DateTime. Consulte la documentación sobre las funciones [startofyear](../query/startofyearfunction.md), [startofmonth](../query/startofmonthfunction.md), [iniciodelasemana](../query/startofweekfunction.md), [startofday](../query/startofdayfunction.md) o [bin](../query/binfunction.md) .

  *PartitionName* `:` `datetime` `=` ( `startofyear` \| `startofmonth` \| `startofweek` \| `startofday` ) `(` *ColumnName*`)`  
  *PartitionName* `:` `datetime` `=` `bin` `(` *ColumnName* `,` *TimeSpan*`)`


<a name="path-format"></a>
*PathFormat*

Formato de ruta de acceso del archivo URI de datos externos, que se puede especificar además de particiones. El formato de ruta de acceso es una secuencia de elementos de partición y separadores de texto:

&nbsp;&nbsp;[*StringSeparator*] *Partición* [*StringSeparator*] [*partición* [*StringSeparator*]...]  

donde *Partition* hace referencia a una partición declarada en la `partition` `by` cláusula y *StringSeparator* es cualquier texto entre comillas.

El prefijo de la ruta de acceso del archivo original se puede construir mediante elementos de partición representados como cadenas y separados con los separadores de texto correspondientes. Para especificar el formato que se usa para representar un valor de partición DateTime, se puede usar la siguiente macro:

&nbsp;&nbsp;`datetime_pattern``(` *DateTimeFormat* `,` *PartitionName*`)`  

donde *DateTimeFormat* se adhiera a la especificación de formato .net, con una extensión que permite incluir especificadores de formato entre corchetes. Por ejemplo, los dos formatos siguientes son equivalentes:

&nbsp;&nbsp;`'year='yyyy'/month='MM`etc`year={yyyy}/month={MM}`

De forma predeterminada, los valores DATETIME se representan con los siguientes formatos:

| Función de partición    | Formato predeterminado |
|-----------------------|----------------|
| `startofyear`         | `yyyy`         |
| `startofmonth`        | `yyyy/MM`      |
| `startofweek`         | `yyyy/MM/dd`   |
| `startofday`          | `yyyy/MM/dd`   |
| `bin(`*Artículo*`, 1d)` | `yyyy/MM/dd`   |
| `bin(`*Artículo*`, 1h)` | `yyyy/MM/dd/HH` |
| `bin(`*Artículo*`, 1m)` | `yyyy/MM/dd/HH/mm` |

Si *PathFormat* se omite de la definición de la tabla externa, se supone que todas las particiones, exactamente en el mismo orden en que están definidas, se `/` separan mediante separador. Las particiones se representan mediante su presentación de cadena predeterminada.

<a name="format"></a>
*Aplique*

El formato de datos, cualquiera de los [formatos de ingesta](../../ingestion-supported-formats.md).

> [!NOTE]
> El uso de una tabla externa para el [escenario de exportación](data-export/export-data-to-an-external-table.md) se limita a los siguientes formatos: `CSV` , `TSV` `JSON` y `Parquet` .

<a name="connection-string"></a>
*StorageConnectionString*

Una o más rutas de acceso para Azure Blob Storage contenedores de blobs o sistemas de archivos Azure Data Lake Store (directorios virtuales o carpetas), incluidas las credenciales.
Consulte [cadenas de conexión de almacenamiento](../api/connection-strings/storage.md) para obtener más información.

> [!TIP]
> Proporcione más que una sola cuenta de almacenamiento para evitar la limitación de almacenamiento al [exportar](data-export/export-data-to-an-external-table.md) grandes cantidades de datos a la tabla externa. La exportación distribuirá las escrituras entre todas las cuentas proporcionadas. 

<a name="properties"></a>
*Propiedades opcionales*

| Propiedad         | Tipo     | Descripción       |
|------------------|----------|-------------------------------------------------------------------------------------|
| `folder`         | `string` | Carpeta de la tabla                                                                     |
| `docString`      | `string` | Cadena que documenta la tabla                                                       |
| `compressed`     | `bool`   | Si se establece, indica si los archivos se comprimen como `.gz` archivos (se usan solo en el [escenario de exportación](data-export/export-data-to-an-external-table.md) ) |
| `includeHeaders` | `string` | En el caso de los archivos CSV o TSV, indica si los archivos contienen un encabezado                     |
| `namePrefix`     | `string` | Si se establece, indica el prefijo de los archivos. En las operaciones de escritura, todos los archivos se escribirán con este prefijo. En las operaciones de lectura, solo se leen los archivos con este prefijo. |
| `fileExtension`  | `string` | Si se establece, indica extensiones de archivo de los archivos. Al escribir, los nombres de los archivos terminarán con este sufijo. En la lectura, solo se leerán los archivos con esta extensión de archivo.           |
| `encoding`       | `string` | Indica cómo se codifica el texto: `UTF8NoBOM` (valor predeterminado) o `UTF8BOM` .             |

> [!TIP]
> Para obtener más información sobre el rol `namePrefix` y `fileExtension` las propiedades que se reproducen en el filtrado de archivos de datos durante la consulta, consulte la sección [lógica de filtrado de archivos](#file-filtering) .
 
<a name="examples"></a>
**Example** 

Una tabla externa sin particiones. Se espera que los archivos de datos se colocan directamente en los contenedores definidos:

```kusto
.create external table ExternalTable (x:long, s:string)  
kind=blob 
dataformat=csv 
( 
   h@'https://storageaccount.blob.core.windows.net/container1;secretKey' 
) 
```

Una tabla externa con particiones por fecha. Se espera que los archivos de fecha se coloquen en directorios de formato de fecha y hora predeterminado `yyyy/MM/dd` :

```kusto
.create external table ExternalTable (Timestamp:datetime, x:long, s:string) 
kind=adl
partition by (Date:datetime = bin(Timestamp, 1d)) 
dataformat=csv 
( 
   h@'abfss://filesystem@storageaccount.dfs.core.windows.net/path;secretKey'
)
```

Una tabla externa con particiones por mes, con el formato de directorio `year=yyyy/month=MM` :

```kusto
.create external table ExternalTable (Timestamp:datetime, x:long, s:string) 
kind=blob 
partition by (Month:datetime = startofmonth(Timestamp)) 
pathformat = (datetime_pattern("'year='yyyy'/month='MM", Month)) 
dataformat=csv 
( 
   h@'https://storageaccount.blob.core.windows.net/container1;secretKey' 
) 
```

Una tabla externa con particiones en primer lugar por nombre de cliente y después por fecha. La estructura de directorios esperada es, por ejemplo, `customer_name=Softworks/2019/02/01` :

```kusto
.create external table ExternalTable (Timestamp:datetime, CustomerName:string) 
kind=blob 
partition by (CustomerNamePart:string = CustomerName, Date:datetime = startofday(Timestamp)) 
pathformat = ("customer_name=" CustomerNamePart "/" Date)
dataformat=csv 
(  
   h@'https://storageaccount.blob.core.windows.net/container1;secretKey' 
)
```

Una tabla externa particionada en primer lugar por el hash de nombre de cliente (módulo 10), a continuación, por fecha. La estructura de directorios esperada es, por ejemplo, `customer_id=5/dt=20190201` . Los nombres de archivo de datos finalizan con la `.txt` extensión:

```kusto
.create external table ExternalTable (Timestamp:datetime, CustomerName:string) 
kind=blob 
partition by (CustomerId:long = hash(CustomerName, 10), Date:datetime = startofday(Timestamp)) 
pathformat = ("customer_id=" CustomerId "/dt=" datetime_pattern("yyyyMMdd", Date)) 
dataformat=csv 
( 
   h@'https://storageaccount.blob.core.windows.net/container1;secretKey'
)
with (fileExtension = ".txt")
```

**Salida de ejemplo**

|TableName|TableType|Carpeta|DocString|Propiedades|ConnectionStrings|Particiones|PathFormat|
|---------|---------|------|---------|----------|-----------------|----------|----------|
|ExternalTable|Blob|ExternalTables|Docs|{"Format": "CSV", "Compressed": false, "CompressionType": null, "FileExtension": null, "IncludeHeaders": "none", "Encoding": null, "NamePrefix": null}|["https://storageaccount.blob.core.windows.net/container1;\*\*\*\*\*\*\*"]|[{"Mod": 10, "Name": "CustomerId", "ColumnName": "CustomerName", "ordinal": 0}, {"Function": "StartOfDay", "Name": "date", "ColumnName": "timestamp", "ordinal": 1}]|"Customer \_ ID =" CustomerID "/DT =" patrón de fecha y hora \_ ("AAAAMMDD", fecha)|

<a name="virtual-columns"></a>
**Columnas virtuales**

Cuando los datos se exportan desde Spark, las columnas de partición (que se especifican en el método del escritor de tramas de datos `partitionBy` ) no se escriben en los archivos de datos. Este proceso evita la duplicación de datos porque los datos ya están presentes en nombres de "carpeta". Por ejemplo, `column1=<value>/column2=<value>/` y Spark puede reconocerlo en el momento de la lectura.

Las tablas externas admiten la siguiente sintaxis para especificar columnas virtuales:

```kusto
.create external table ExternalTable (EventName:string, Revenue:double)  
kind=blob  
partition by (CustomerName:string, Date:datetime)  
pathformat = ("customer=" CustomerName "/date=" datetime_pattern("yyyyMMdd", Date))  
dataformat=parquet
( 
   h@'https://storageaccount.blob.core.windows.net/container1;secretKey'
)
```

<a name="file-filtering"></a>
**Lógica de filtrado de archivos**

Al consultar una tabla externa, el motor de consultas mejora el rendimiento al filtrar los archivos de almacenamiento externo irrelevantes. A continuación se describe el proceso de iteración en archivos y la decisión sobre si se debe procesar un archivo.

1. Cree un patrón de URI que represente un lugar donde se encuentren los archivos. Inicialmente, el patrón de URI es igual a una cadena de conexión proporcionada como parte de la definición de tabla externa. Si hay particiones definidas, se representan mediante *[PathFormat](#path-format)* y, a continuación, se anexan al patrón de URI.

2. En el caso de todos los archivos que se encuentran en los patrones de URI creados, compruebe lo siguiente:

   * Los valores de partición coinciden con predicados usados en una consulta.
   * El nombre del BLOB comienza por `NamePrefix` , si se define una propiedad de este tipo.
   * El nombre del BLOB termina con `FileExtension` , si se define una propiedad de este tipo.

Una vez que se cumplen todas las condiciones, el motor de consultas captura y procesa el archivo.

> [!NOTE]
> El patrón de URI inicial se crea con valores de predicado de consulta. Esto funciona mejor para un conjunto limitado de valores de cadena, así como para intervalos de tiempo cerrados. 

## <a name="show-external-table-artifacts"></a>. Mostrar artefactos de tabla externa

Devuelve una lista de todos los archivos que se procesarán al consultar una tabla externa determinada.

> [!NOTE]
> La operación requiere [permiso de usuario de base de datos](../management/access-control/role-based-authorization.md).

**Sintaxis:** 

`.show``external` `table` *TableName* `artifacts` [ `limit` *MaxResults*]

donde *MaxResults* es un parámetro opcional, que se puede establecer para limitar el número de resultados.

**Salida**

| Parámetro de salida | Tipo   | Descripción                       |
|------------------|--------|-----------------------------------|
| URI              | string | URI del archivo de datos de almacenamiento externo |

> [!TIP]
> La iteración en todos los archivos a los que hace referencia una tabla externa puede ser bastante costosa, en función del número de archivos. Asegúrese de usar el `limit` parámetro si solo desea ver algunos ejemplos de URI.

**Ejemplos:**

```kusto
.show external table T artifacts
```

**Salida:**

| Identificador URI                                                                     |
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

| Nombre     | Tipo | Asignación                                                           |
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

| Nombre     | Tipo | Asignación                                                                |
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

| Nombre     | Tipo | Asignación                                                                         |
|----------|------|---------------------------------------------------------------------------------|
| mapping1 | JSON | [{"ColumnName": "RowNumber", "ColumnType": "", "propiedades": {"ruta de acceso": "$. RowNumber"}}, {"ColumnName": "ROWGUID", "ColumnType": "", "propiedades": {"ruta de acceso": "$. ROWGUID"}}] |

## <a name="drop-external-table-mapping"></a>. quitar asignación de tabla externa

`.drop``external` `table` *ExternalTableName* `json` `mapping` *MappingName* 

Quita la asignación de la base de datos.
 
**Ejemplo** 
 
```kusto
.drop external table MyExternalTable JSON mapping "Mapping1" 
```
## <a name="next-steps"></a>Pasos siguientes

* [Comandos de control general de tabla externa](externaltables.md)
* [Creación y modificación de tablas SQL externas](external-sql-tables.md)
