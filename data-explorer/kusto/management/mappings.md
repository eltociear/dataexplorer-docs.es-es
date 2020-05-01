---
title: 'Asignaciones de datos: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las asignaciones de datos en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: 2a3b402c04d5d1af85b2c2a042a23fbade7e2524
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617653"
---
# <a name="data-mappings"></a>Asignaciones de datos

Las asignaciones de datos se usan durante la ingesta para asignar datos entrantes a las columnas dentro de las tablas de Kusto.

Kusto admite distintos tipos de asignaciones: `row-oriented` (CSV, JSON y Avro) y `column-oriented` (parquet).

Cada elemento de la lista de asignación se construye a partir de tres propiedades:

|Propiedad|Descripción|
|----|--|
|`column`|Nombre de la columna de destino en la tabla Kusto|
|`datatype`| Opta DataType con la que se creará la columna asignada si aún no existe en la tabla Kusto|
|`Properties`|Opta Contenedor de propiedades que contiene las propiedades específicas de cada asignación, tal y como se describe en cada sección siguiente.


Todas las asignaciones se pueden [crear previamente](create-ingestion-mapping-command.md) y se puede hacer referencia a ellas desde el comando de `ingestionMappingReference` introducción mediante parámetros.

## <a name="csv-mapping"></a>Asignación de CSV

Cuando el archivo de código fuente es un CSV (o cualquier formato separado por delimitador) y su esquema no coincide con el esquema de la tabla Kusto actual, una asignación CSV se asigna desde el esquema de archivo al esquema de la tabla Kusto. Si la tabla no existe en Kusto, se creará de acuerdo con esta asignación. Si faltan algunos campos de la asignación en la tabla, se agregarán. 

La asignación de CSV se puede aplicar en todos los formatos separados por delimitadores: CSV, TSV, PSV, SCSV y SOHsv.

Cada elemento de la lista describe una asignación para una columna específica y puede contener las siguientes propiedades:

|Propiedad|Descripción|
|----|--|
|`ordinal`|El número de orden de la columna en CSV|
|`constantValue`|Opta Valor constante que se va a utilizar para una columna en lugar de un valor dentro del CSV.|

> [!NOTE]
> `Ordinal`y `ConstantValue` son mutuamente excluyentes.

### <a name="example-of-the-csv-mapping"></a>Ejemplo de asignación de CSV

``` json
[
  { "column" : "rownumber", "Properties":{"Ordinal":"0"}},
  { "column" : "rowguid",   "Properties":{"Ordinal":"1"}},
  { "column" : "xdouble",   "Properties":{"Ordinal":"2"}},
  { "column" : "xbool",     "Properties":{"Ordinal":"3"}},
  { "column" : "xint32",    "Properties":{"Ordinal":"4"}},
  { "column" : "xint64",    "Properties":{"Ordinal":"5"}},
  { "column" : "xdate",     "Properties":{"Ordinal":"6"}},
  { "column" : "xtext",     "Properties":{"Ordinal":"7"}},
  { "column" : "const_val", "Properties":{"ConstValue":"Sample: constant value"}}
]
```

> [!NOTE]
> Cuando se proporciona la asignación anterior como parte del comando `.ingest` de control, se serializa como una cadena JSON.

* Cuando se [crea previamente](create-ingestion-mapping-command.md) la asignación anterior, se puede hacer referencia a ella en `.ingest` el comando de control:

```kusto
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="csv", 
        ingestionMappingReference = "Mapping1"
    )
```

* Cuando se proporciona la asignación anterior como parte del comando `.ingest` de control, se serializa como una cadena JSON:

```kusto
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="csv", 
        ingestionMapping = 
        "["
            "{\"column\":\"rownumber\",\"Properties\":{\"Ordinal\":0}},"
            "{\"column\":\"rowguid\",  \"Properties\":{\"Ordinal\":1}}"
        "]" 
    )
```

**Nota:** Actualmente se admite el siguiente formato de `Properties` asignación, sin la bolsa de propiedades, pero puede estar en desuso en el futuro.

```kusto
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="csv", 
        ingestionMapping = 
        "["
            "{\"column\":\"rownumber\",\"Ordinal\": 0},"
            "{\"column\":\"rowguid\",  \"Ordinal\": 1}"
        "]" 
    )
```

## <a name="json-mapping"></a>Asignación de JSON

Cuando el archivo de código fuente está en formato JSON, el contenido del archivo se asigna a la tabla Kusto. La tabla debe existir en la base de datos Kusto a menos que se especifique un tipo de datos válido para todas las columnas asignadas. Las columnas asignadas en la asignación JSON deben existir en la tabla Kusto a menos que se especifique un tipo de usuario para todas las columnas que no existen.

Cada elemento de la lista describe una asignación para una columna específica y puede contener las siguientes propiedades: 

|Propiedad|Descripción|
|----|--|
|`path`|Si comienza por `$`: ruta de acceso JSON al campo que se convertirá en el contenido de la columna del documento JSON (la ruta de acceso JSON que denota todo `$`el documento es). Si el valor no se inicia con `$`: se utiliza un valor constante.|
|`transform`|Opta Transformación que se debe aplicar en el contenido con [transformaciones de asignación](#mapping-transformations).|

### <a name="example-of-json-mapping"></a>Ejemplo de asignación de JSON

```json
[
  { "column" : "rownumber",   "Properties":{"Path":"$.rownumber"}}, 
  { "column" : "rowguid",     "Properties":{"Path":"$.rowguid"}}, 
  { "column" : "xdouble",     "Properties":{"Path":"$.xdouble"}}, 
  { "column" : "xbool",       "Properties":{"Path":"$.xbool"}}, 
  { "column" : "xint32",      "Properties":{"Path":"$.xint32"}}, 
  { "column" : "xint64",      "Properties":{"Path":"$.xint64"}}, 
  { "column" : "xdate",       "Properties":{"Path":"$.xdate"}}, 
  { "column" : "xtext",       "Properties":{"Path":"$.xtext"}}, 
  { "column" : "location",    "Properties":{"transform":"SourceLocation"}}, 
  { "column" : "lineNumber",  "Properties":{"transform":"SourceLineNumber"}}, 
  { "column" : "full_record", "Properties":{"Path":"$"}}
]
```

> [!NOTE]
> Cuando se proporciona la asignación anterior como parte del comando `.ingest` de control, se serializa como una cadena JSON.

```kusto
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="json", 
        ingestionMappingReference = "Mapping1"
    )
```

**Nota:** Actualmente se admite el siguiente formato de `Properties` asignación, sin la bolsa de propiedades, pero puede estar en desuso en el futuro.

```kusto
.ingest into Table123 (@"source1", @"source2") 
  with 
  (
      format = "json", 
      ingestionMapping = 
      "["
        "{\"column\":\"rownumber\",\"path\":\"$.rownumber\"},"
        "{\"column\":\"rowguid\",  \"path\":\"$.rowguid\"}"
      "]"
  )
```
    
## <a name="avro-mapping"></a>Asignación de Avro

Cuando el archivo de código fuente está en formato Avro, el contenido del archivo Avro se asigna a la tabla Kusto. La tabla debe existir en la base de datos Kusto a menos que se especifique un tipo de datos válido para todas las columnas asignadas. Las columnas asignadas en la asignación de Avro deben existir en la tabla Kusto a menos que se especifique un tipo de usuario para todas las columnas no existentes.

Cada elemento de la lista describe una asignación para una columna específica y puede contener las siguientes propiedades: 

|Propiedad|Descripción|
|----|--|
|`Field`|Nombre del campo en el registro Avro.|
|`Path`|Alternativa al uso `field` de que permite tomar la parte interna de un campo de registro de Avro, si es necesario. El valor denota una ruta de acceso JSON de la raíz del registro. Vea las notas siguientes para obtener más información. |
|`transform`|Opta Transformación que se debe aplicar en el contenido con [transformaciones admitidas](#mapping-transformations).|

**Notas**
>[!NOTE]
> * `field`y `path` no se pueden usar juntos, solo se permite uno. 
> * `path`no puede apuntar `$` solo a la raíz, debe tener al menos un nivel de ruta de acceso.

Las dos alternativas siguientes son iguales:

``` json
[
  {"column": "rownumber", "Properties":{"Path":"$.RowNumber"}}
]
```

``` json
[
  {"column": "rownumber", "Properties":{"Field":"RowNumber"}}
]
```

### <a name="example-of-the-avro-mapping"></a>Ejemplo de la asignación de AVRO

``` json
[
  {"column": "rownumber", "Properties":{"Field":"rownumber"}},
  {"column": "rowguid",   "Properties":{"Field":"rowguid"}},
  {"column": "xdouble",   "Properties":{"Field":"xdouble"}},
  {"column": "xboolean",  "Properties":{"Field":"xboolean"}},
  {"column": "xint32",    "Properties":{"Field":"xint32"}},
  {"column": "xint64",    "Properties":{"Field":"xint64"}},
  {"column": "xdate",     "Properties":{"Field":"xdate"}},
  {"column": "xtext",     "Properties":{"Field":"xtext"}}
]
``` 

> [!NOTE]
> Cuando se proporciona la asignación anterior como parte del comando `.ingest` de control, se serializa como una cadena JSON.

```kusto
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="avro", 
        ingestionMappingReference = "Mapping1"
    )
```

**Nota:** Actualmente se admite el siguiente formato de `Properties` asignación, sin la bolsa de propiedades, pero puede estar en desuso en el futuro.

```kusto
.ingest into Table123 (@"source1", @"source2") 
  with 
  (
      format = "avro", 
      ingestionMapping = 
      "["
        "{\"column\":\"rownumber\",\"field\":\"rownumber\"},"
        "{\"column\":\"rowguid\",  \"field\":\"rowguid\"}"
      "]"
  )
```

## <a name="parquet-mapping"></a>Asignación de parquet

Cuando el archivo de código fuente está en formato parquet, el contenido del archivo se asigna a la tabla Kusto. La tabla debe existir en la base de datos Kusto a menos que se especifique un tipo de datos válido para todas las columnas asignadas. Las columnas asignadas en la asignación parquet deben existir en la tabla Kusto, a menos que se especifique un tipo de usuario para todas las columnas no existentes.

Cada elemento de la lista describe una asignación para una columna específica y puede contener las siguientes propiedades:

|Propiedad|Descripción|
|----|--|
|`path`|Si comienza por `$`: ruta de acceso JSON al campo que se convertirá en el contenido de la columna en el documento parquet (la ruta de acceso JSON que denota `$`todo el documento es). Si el valor no se inicia con `$`: se utiliza un valor constante.|
|`transform`|Opta [asignación de transformaciones](#mapping-transformations) que se deben aplicar en el contenido.


### <a name="example-of-the-parquet-mapping"></a>Ejemplo de asignación parquet

```json
[
  { "column" : "rownumber",   "Properties":{"Path":"$.rownumber"}}, 
  { "column" : "xdouble",     "Properties":{"Path":"$.xdouble"}}, 
  { "column" : "xbool",       "Properties":{"Path":"$.xbool"}}, 
  { "column" : "xint64",      "Properties":{"Path":"$.xint64"}}, 
  { "column" : "xdate",       "Properties":{"Path":"$.xdate"}}, 
  { "column" : "xtext",       "Properties":{"Path":"$.xtext"}}, 
  { "column" : "location",    "Properties":{"transform":"SourceLocation"}}, 
  { "column" : "lineNumber",  "Properties":{"transform":"SourceLineNumber"}}, 
  { "column" : "full_record", "Properties":{"Path":"$"}}
]
```      

> [!NOTE]
> Cuando se proporciona la asignación anterior como parte del comando `.ingest` de control, se serializa como una cadena JSON.

* Cuando se [crea previamente](create-ingestion-mapping-command.md) la asignación anterior, se puede hacer referencia a ella en `.ingest` el comando de control:

```kusto
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="parquet", 
        ingestionMappingReference = "Mapping1"
    )
```

* Cuando se proporciona la asignación anterior como parte del comando `.ingest` de control, se serializa como una cadena JSON:

```kusto
.ingest into Table123 (@"source1", @"source2") 
  with 
  (
      format = "parquet", 
      ingestionMapping = 
      "["
        "{\"column\":\"rownumber\",\"Properties\":{\"Path\":\"$.rownumber\"}},"
        "{\"column\":\"rowguid\",  \"Properties\":{\"Path\":\"$.rowguid\"}}"
      "]"
  )
```

## <a name="orc-mapping"></a>Asignación de ORC

Cuando el archivo de código fuente está en formato ORC, el contenido del archivo se asigna a la tabla Kusto. La tabla debe existir en la base de datos Kusto a menos que se especifique un tipo de datos válido para todas las columnas asignadas. Las columnas asignadas en la asignación Orc deben existir en la tabla Kusto, a menos que se especifique un tipo de usuario para todas las columnas no existentes.

Cada elemento de la lista describe una asignación para una columna específica y puede contener las siguientes propiedades:

|Propiedad|Descripción|
|----|--|
|`path`|Si comienza por `$`: ruta de acceso JSON al campo que se convertirá en el contenido de la columna en el documento ORC (la ruta de acceso JSON que denota `$`todo el documento es). Si el valor no se inicia con `$`: se utiliza un valor constante.|
|`transform`|Opta [asignación de transformaciones](#mapping-transformations) que se deben aplicar en el contenido.

### <a name="example-of-orc-mapping"></a>Ejemplo de asignación de ORC

```json
[
  { "column" : "rownumber",   "Properties":{"Path":"$.rownumber"}}, 
  { "column" : "xdouble",     "Properties":{"Path":"$.xdouble"}}, 
  { "column" : "xbool",       "Properties":{"Path":"$.xbool"}}, 
  { "column" : "xint64",      "Properties":{"Path":"$.xint64"}}, 
  { "column" : "xdate",       "Properties":{"Path":"$.xdate"}}, 
  { "column" : "xtext",       "Properties":{"Path":"$.xtext"}}, 
  { "column" : "location",    "Properties":{"transform":"SourceLocation"}}, 
  { "column" : "lineNumber",  "Properties":{"transform":"SourceLineNumber"}}, 
  { "column" : "full_record", "Properties":{"Path":"$"}}
]
```      

> [!NOTE]
> Cuando se proporciona la asignación anterior como parte del comando `.ingest` de control, se serializa como una cadena JSON.

```kusto
.ingest into Table123 (@"source1", @"source2") 
  with 
  (
      format = "orc", 
      ingestionMapping = 
      "["
        "{\"column\":\"rownumber\",\"Properties\":{\"Path\":\"$.rownumber\"}},"
        "{\"column\":\"rowguid\",  \"Properties\":{\"Path\":\"$.rowguid\"}}"
      "]"
  )
```

## <a name="mapping-transformations"></a>Asignación de transformaciones

Algunas de las asignaciones de formato de datos (parquet, JSON y Avro) admiten transformaciones de tiempo de ingesta sencillas y útiles. Si el escenario requiere un procesamiento más complejo en el momento de la ingesta, use [actualizar directiva](update-policy.md), que permite definir el procesamiento ligero mediante la expresión de KQL.

|Transformación dependiente de la ruta de acceso|Descripción|Condiciones|
|--|--|--|
|`PropertyBagArrayToDictionary`|Transforma una matriz JSON de propiedades (por ejemplo, {eventos: [{"N1": "v1"}, {"N2": "V2"}]}) al diccionario y lo serializa en un documento JSON válido (por ejemplo, {"N1": "v1", "N2": "V2"}).|Solo se puede aplicar cuando `path` se usa|
|`GetPathElement(index)`|Extrae un elemento de la ruta de acceso dada según el índice especificado (por ejemplo, ruta de acceso: $. a. b. c, GetPathElement (0) = = "c", GetPathElement (-1) = = "b", cadena de tipo|Solo se puede aplicar cuando `path` se usa|
|`SourceLocation`|Nombre del artefacto de almacenamiento que proporcionó los datos, tipo cadena (por ejemplo, el campo "BaseUri" del BLOB).|
|`SourceLineNumber`|Desplazamiento con respecto a ese artefacto de almacenamiento, tipo Long (a partir de ' 1 ' y incremento por registro nuevo).|
|`DateTimeFromUnixSeconds`|Convierte el número que representa el tiempo de UNIX (segundos desde 1970-01-01) en una cadena de fecha y hora UTC|
|`DateTimeFromUnixMilliseconds`|Convierte el número que representa el tiempo de UNIX (en milisegundos desde 1970-01-01) en una cadena de fecha y hora UTC|
|`DateTimeFromUnixMicroseconds`|Convierte el número que representa el tiempo de UNIX (en microsegundos desde 1970-01-01) en una cadena de fecha y hora UTC.|
|`DateTimeFromUnixNanoseconds`|Convierte el número que representa el tiempo de UNIX (nanosegundos desde 1970-01-01) en una cadena de fecha y hora UTC.|
