---
title: El comando .ingest into (pull data from storage) - Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe el comando .ingest into (pull data from storage) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 1f304f9dc064094c6d55cb32f3fb453f32114979
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521444"
---
# <a name="the-ingest-into-command-pull-data-from-storage"></a>El comando .ingest into (pull data from storage)

El `.ingest into` comando ingesta los datos en una tabla "tirando" de los datos de uno o más artefactos de almacenamiento en la nube.
Por ejemplo, el comando puede recuperar 1000 blobs con formato CSV de Azure Blob Storage, analizarlos e ingerirnos juntos en una única tabla de destino.
Los datos se anexan a la tabla sin afectar a los registros existentes y sin modificar el esquema de la tabla.

**Sintaxis**

`.ingest`[`async` `into` `table` ] *TableName* *SourceDataLocator* `with` `(` [ *IngestionPropertyName* `=` *IngestionPropertyValue* [`,` ...] `)`]

**Argumentos**

* `async`: Si se especifica, el comando volverá inmediatamente y continuará la ingesta en segundo plano. Los resultados del comando `OperationId` incluirán un valor que, a continuación, se puede utilizar con el `.show operation` comando para recuperar el estado de finalización de la ingesta y los resultados.
  
* *TableName*: el nombre de la tabla en la que se ingiren datos.
  El nombre de la tabla siempre es relativo a la base de datos en contexto y su esquema es el esquema que se asumirá para los datos si no se proporciona ningún objeto de asignación de esquema.

* *SourceDataLocator*: un `string`literal de tipo , o una lista `(` delimitada por comas de estos literales rodeados por caracteres y `)` caracteres, que indica los artefactos de almacenamiento que contienen los datos que se van a extraer. Consulte Cadenas de [conexión de almacenamiento](../../api/connection-strings/storage.md).

> [!NOTE]
> Se recomienda encarecidamente usar literales de [cadena ofuscados](../../query/scalar-data-types/string.md#obfuscated-string-literals) para *SourceDataPointer* que incluya credenciales reales en él.
> El servicio se asegurará de eliminar las credenciales en sus seguimientos internos, mensajes de error, etc.

* *IngestionPropertyName*, *IngestionPropertyValue*: cualquier número de propiedades de [ingesta](https://docs.microsoft.com/azure/data-explorer/ingestion-properties) que afecten al proceso de ingesta.

**Resultados**

El resultado del comando es una tabla con tantos registros como particiones de datos ("extensiones") generadas por el comando.
Si no se han generado particiones de datos, se devuelve un único registro con un identificador de extensión vacío (de valor cero).

|Nombre       |Tipo      |Descripción                                                                |
|-----------|----------|---------------------------------------------------------------------------|
|ExtentId   |`guid`    |Identificador único de la partición de datos generada por el comando.|
|ItemLoaded |`string`  |Uno o varios artefactos de almacenamiento relacionados con este registro.             |
|Duration   |`timespan`|Cuánto tiempo se tardó en realizar la ingestión.                                     |
|HasErrors  |`bool`    |Si este registro representa un error de ingesta o no.                |
|OperationId|`guid`    |Un identificador único que representa la operación. Se puede utilizar `.show operation` con el comando.|

**Comentarios:**

Este comando no modifica el esquema de la tabla en la que se ingesta.
Si es necesario, los datos se "coaccionan" en este esquema durante la ingesta, no al revés (se omiten columnas adicionales y las columnas que faltan se tratan como valores nulos).

**Ejemplos**

En el ejemplo siguiente se indica al motor que lea dos blobs de `T`Azure Blob Storage como archivos CSV e ingleen su contenido en la tabla. Representa `...` una firma de acceso compartido (SAS) de Azure Storage que proporciona acceso de lectura a cada blob. Tenga en cuenta también el uso `h` de cadenas ofuscadas (la parte frontal de los valores de cadena) para asegurarse de que la SAS nunca se registra.

```kusto
.ingest into table T (
    h'https://contoso.blob.core.windows.net/container/file1.csv?...',
    h'https://contoso.blob.core.windows.net/container/file2.csv?...'
)
```

El siguiente ejemplo es para la ingesta de datos de Azure Data Lake Storage Gen 2 (ADLSv2). Las credenciales que`...`se usan aquí ( ) son las credenciales de la cuenta de almacenamiento (clave compartida) y usamos ofuscación de cadenas solo para la parte secreta de la cadena de conexión.

```kusto
.ingest into table T (
  'abfss://myfilesystem@contoso.dfs.core.windows.net/path/to/file1.csv;'
    h'...'
)
```

En el ejemplo siguiente se ingesta un único archivo de Azure Data Lake Storage (ADLS).
Utiliza las credenciales del usuario para tener acceso a ADLS (por lo que no es necesario tratar el URI de almacenamiento como que contiene un secreto). También muestra cómo especificar propiedades de ingesta.

```kusto
.ingest into table T ('adl://contoso.azuredatalakestore.net/Path/To/File/file1.ext;impersonate')
  with (format='csv')
```

