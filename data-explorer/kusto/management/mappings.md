---
title: 'Asignaciones de datos: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen las asignaciones de datos en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: 0d94815eedfd551a09a979c57c68baf125abec40
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520781"
---
# <a name="data-mappings"></a>Asignaciones de datos

Las asignaciones de datos se utilizan durante la ingesta para asignar datos entrantes a columnas dentro de tablas Kusto.

Kusto admite diferentes tipos de `row-oriented` asignaciones, tanto (CSV, `column-oriented` JSON y AVRO) como (Parquet).

Cada elemento de la lista de asignación se construye a partir de tres propiedades:

|Propiedad|Descripción|
|----|--|
|`column`|Nombre de columna de destino en la tabla Kusto|
|`datatype`| (Opcional) Tipo de datos con el que crear la columna asignada si aún no existe en la tabla Kusto|
|`Properties`|(Opcional) Bolsa de propiedades que contiene propiedades específicas para cada asignación como se describe en cada sección siguiente.


Todas las asignaciones se pueden [crear previamente](create-ingestion-mapping-command.md) y `ingestionMappingReference` se puede hacer referencia a ellas desde el comando ingest mediante parámetros.

## <a name="csv-mapping"></a>Mapeo CSV

Cuando el archivo de origen es un CSV (o cualquier formato separado por delímetros) y su esquema no coincide con el esquema de tabla Kusto actual, una asignación CSV se asigna desde el esquema de archivo al esquema de tabla Kusto. Si la tabla no existe en Kusto, se creará de acuerdo con esta asignación. Si faltan algunos campos de la asignación en la tabla, se agregarán. 

La asignación CSV se puede aplicar en todos los formatos separados por delimitadores: CSV, TSV, PSV, SCSV y SOHsv.

Cada elemento de la lista describe una asignación para una columna específica y puede contener las siguientes propiedades:

|Propiedad|Descripción|
|----|--|
|`ordinal`|El número de orden de columna en CSV|
|`constantValue`|(Opcional) El valor constante que se usará para una columna en lugar de algún valor dentro del CSV|

> [!NOTE]
> `Ordinal`y `ConstantValue` son mutuamente excluyentes.

### <a name="example-of-the-csv-mapping"></a>Ejemplo de la asignación CSV

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
> Cuando la asignación anterior se `.ingest` proporciona como parte del comando de control, se serializa como una cadena JSON.

* Cuando se [crea previamente](create-ingestion-mapping-command.md) la asignación `.ingest` anterior, se puede hacer referencia a ella en el comando de control:
```
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="csv", 
        ingestionMappingReference = "Mapping1"
    )
```

* Cuando la asignación anterior se `.ingest` proporciona como parte del comando de control, se serializa como una cadena JSON:

```
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

**Nota:** El siguiente formato de `Properties` asignación, sin la propiedad-bolsa, se admite actualmente, pero puede estar en desuso en el futuro.

```
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

## <a name="json-mapping"></a>Asignación JSON

Cuando el archivo de origen está en formato JSON, el contenido del archivo se asigna a la tabla Kusto. La tabla debe existir en la base de datos Kusto a menos que se especifique un tipo de datos válido para todas las columnas asignadas. Las columnas asignadas en la asignación JSON deben existir en la tabla Kusto a menos que se especifique un tipo de datos para todas las columnas no existentes.

Cada elemento de la lista describe una asignación para una columna específica y puede contener las siguientes propiedades: 

|Propiedad|Descripción|
|----|--|
|`path`|Si comienza `$`con : ruta JSON al campo que se convertirá en el contenido de la `$`columna en el documento JSON (ruta JSON que denota todo el documento es ). Si el valor no `$`comienza con : se utiliza un valor constante.|
|`transform`|(Opcional) Transformación que se debe aplicar en el contenido con transformaciones de [asignación.](#mapping-transformations)|

### <a name="example-of-json-mapping"></a>Ejemplo de mapeo JSON

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
> Cuando la asignación anterior se `.ingest` proporciona como parte del comando de control, se serializa como cadena JSON.

```
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="json", 
        ingestionMappingReference = "Mapping1"
    )
```

**Nota:** El siguiente formato de `Properties` asignación, sin la propiedad-bolsa, se admite actualmente, pero puede estar en desuso en el futuro.

```
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
    
## <a name="avro-mapping"></a>Mapeo Avro

Cuando el archivo de origen está en formato Avro, el contenido del archivo Avro se asigna a la tabla Kusto. La tabla debe existir en la base de datos Kusto a menos que se especifique un tipo de datos válido para todas las columnas asignadas. Las columnas asignadas en la asignación aVro deben existir en la tabla Kusto a menos que se especifique un tipo de datos para todas las columnas no existentes.

Cada elemento de la lista describe una asignación para una columna específica y puede contener las siguientes propiedades: 

|Propiedad|Descripción|
|----|--|
|`Field`|El nombre del campo en el registro Avro.|
|`Path`|Alternativa al `field` uso que permite tomar la parte interna de un campo de registro Avro, si es necesario. El valor denota una ruta JSON desde la raíz del registro. Consulte las Notas a continuación para obtener más información. |
|`transform`|(Opcional) Transformación que se debe aplicar en el contenido con [transformaciones admitidas.](#mapping-transformations)|

**Notas**
>[!NOTE]
> * `field`y `path` no se pueden usar juntos, sólo se permite uno. 
> * `path`no puede `$` apuntar a la raíz solamente, debe tener al menos un nivel de trayectoria.

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

### <a name="example-of-the-avro-mapping"></a>Ejemplo de la asignación AVRO

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
> Cuando la asignación anterior se `.ingest` proporciona como parte del comando de control, se serializa como cadena JSON.

```
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="avro", 
        ingestionMappingReference = "Mapping1"
    )
```

**Nota:** El siguiente formato de `Properties` asignación, sin la propiedad-bolsa, se admite actualmente, pero puede estar en desuso en el futuro.

```
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

## <a name="parquet-mapping"></a>Mapeo de parquet

Cuando el archivo de origen está en formato Parquet, el contenido del archivo se asigna a la tabla Kusto. La tabla debe existir en la base de datos Kusto a menos que se especifique un tipo de datos válido para todas las columnas asignadas. Las columnas asignadas en la asignación parquet deben existir en la tabla Kusto a menos que se especifique un tipo de datos para todas las columnas no existentes.

Cada elemento de la lista describe una asignación para una columna específica y puede contener las siguientes propiedades:

|Propiedad|Descripción|
|----|--|
|`path`|Si comienza `$`con : ruta JSON al campo que se convertirá en el contenido de la `$`columna en el documento Parquet (ruta JSON que denota todo el documento es ). Si el valor no `$`comienza con : se utiliza un valor constante.|
|`transform`|(Opcional) [transformaciones](#mapping-transformations) de asignación que se deben aplicar en el contenido.


### <a name="example-of-the-parquet-mapping"></a>Ejemplo de la asignación de Parquet

``` json
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
> Cuando la asignación anterior se `.ingest` proporciona como parte del comando de control, se serializa como una cadena JSON.

* Cuando se [crea previamente](create-ingestion-mapping-command.md) la asignación `.ingest` anterior, se puede hacer referencia a ella en el comando de control:

```
.ingest into Table123 (@"source1", @"source2")
    with 
    (
        format="parquet", 
        ingestionMappingReference = "Mapping1"
    )
```

* Cuando la asignación anterior se `.ingest` proporciona como parte del comando de control, se serializa como una cadena JSON:

```
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

## <a name="orc-mapping"></a>Mapeo de orcos

Cuando el archivo de origen está en formato Orc, el contenido del archivo se asigna a la tabla Kusto. La tabla debe existir en la base de datos Kusto a menos que se especifique un tipo de datos válido para todas las columnas asignadas. Las columnas asignadas en la asignación orc deben existir en la tabla Kusto a menos que se especifique un tipo de datos para todas las columnas no existentes.

Cada elemento de la lista describe una asignación para una columna específica y puede contener las siguientes propiedades:

|Propiedad|Descripción|
|----|--|
|`path`|Si comienza `$`con : ruta JSON al campo que se convertirá en el contenido de la `$`columna en el documento Orc (ruta JSON que denota todo el documento es ). Si el valor no `$`comienza con : se utiliza un valor constante.|
|`transform`|(Opcional) [transformaciones](#mapping-transformations) de asignación que se deben aplicar en el contenido.

### <a name="example-of-orc-mapping"></a>Ejemplo de mapeo orco

``` json
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
> Cuando la asignación anterior se `.ingest` proporciona como parte del comando de control, se serializa como una cadena JSON.

```
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

## <a name="mapping-transformations"></a>Transformación de mapas

Algunas de las asignaciones de formato de datos (Parquet, JSON y Avro) admiten transformaciones simples y útiles en tiempo de ingesta. Cuando el escenario requiera un procesamiento más complejo en el momento de la ingesta, utilice la [directiva Update](update-policy.md), que permite definir el procesamiento ligero mediante la expresión KQL.

|Transformación dependiente del trayecto|Descripción|Condiciones|
|--|--|--|
|`PropertyBagArrayToDictionary`|Transforma la matriz JSON de propiedades (por ejemplo, "eventos:["n1":"v1","n2":"v2"-]-) en un diccionario y lo serializa en un documento JSON válido (por ejemplo, "n1":"v1","n2":"v2"-).|Sólo se puede `path` aplicar cuando se utiliza|
|`GetPathElement(index)`|Extrae un elemento de la ruta de acceso dada de acuerdo con el índice dado (por ejemplo, Path: $.a.b.c, GetPathElement(0) á "c", GetPathElement(-1) á "b", cadena de tipo|Sólo se puede `path` aplicar cuando se utiliza|
|`SourceLocation`|Nombre del artefacto de almacenamiento que proporcionó los datos, escriba string (por ejemplo, el campo "BaseUri" del blob).|
|`SourceLineNumber`|Desplazamiento relativo a ese artefacto de almacenamiento, escriba long (empezando por '1' e incrementando por nuevo registro).|
|`DateTimeFromUnixSeconds`|Convierte el número que representa unix-time (segundos desde 1970-01-01) en cadena de fecha y hora UTC|
|`DateTimeFromUnixMilliseconds`|Convierte el número que representa unix-time (milisegundos desde 1970-01-01) en cadena de fecha y hora UTC|
|`DateTimeFromUnixMicroseconds`|Convierte el número que representa unix-time (microsegundos desde 1970-01-01) en cadena de fecha y hora UTC|
|`DateTimeFromUnixNanoseconds`|Convierte el número que representa unix-time (nanosegundos desde 1970-01-01) en cadena de fecha y hora UTC|