---
title: El comando. ingerir en (extraer datos del almacenamiento)-Azure Explorador de datos | Microsoft Docs
description: En este artículo se describe el comando. ingerir en el comando (extraer datos de almacenamiento) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 05d62aaa7b123f7f6d02b784402fd06335e155b2
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373422"
---
# <a name="the-ingest-into-command-pull-data-from-storage"></a>El comando. ingerir en (extraer datos del almacenamiento)

El `.ingest into` comando ingeri los datos en una tabla mediante la "extracción" de los datos de uno o varios artefactos de almacenamiento en la nube.
Por ejemplo, el comando puede recuperar los blobs con formato CSV 1000 de Azure Blob Storage, analizarlos y ingerirlos juntos en una sola tabla de destino.
Los datos se anexan a la tabla sin afectar a los registros existentes y sin modificar el esquema de la tabla.

**Sintaxis**

`.ingest`[ `async` ] `into` `table` *TableName* *SourceDataLocator* [ `with` `(` *IngestionPropertyName* `=` *IngestionPropertyValue* [ `,` ...] `)` ]

**Argumentos**

* `async`: Si se especifica, el comando se devolverá inmediatamente y continuará la ingesta en segundo plano. Los resultados del comando incluirán un `OperationId` valor que se puede usar con el `.show operation` comando para recuperar el estado y los resultados de la finalización de la ingesta.
  
* *TableName*: nombre de la tabla a la que se van a ingerir datos.
  El nombre de la tabla siempre es relativo a la base de datos en contexto, y su esquema es el esquema que se asumirá para los datos si no se proporciona ningún objeto de asignación de esquema.

* *SourceDataLocator*: un literal de tipo `string` , o una lista delimitada por comas de estos literales rodeados por `(` `)` caracteres y, que indican los artefactos de almacenamiento que contienen los datos que se van a extraer. Vea [cadenas de conexión de almacenamiento](../../api/connection-strings/storage.md).

> [!NOTE]
> Se recomienda encarecidamente el uso de [literales de cadena ofuscados](../../query/scalar-data-types/string.md#obfuscated-string-literals) para el *SourceDataPointer* que incluye las credenciales reales en él.
> El servicio se asegurará de borrar las credenciales en sus seguimientos internos, mensajes de error, etc.

* *IngestionPropertyName*, *IngestionPropertyValue*: cualquier número de [propiedades de ingesta](../../../ingestion-properties.md) que afecten al proceso de ingesta.

**Resultados**

El resultado del comando es una tabla con tantos registros como particiones de datos ("extensiones") generados por el comando.
Si no se han generado particiones de datos, se devuelve un único registro con un identificador de extensión vacío (de valor cero).

|Nombre       |Tipo      |Descripción                                                                |
|-----------|----------|---------------------------------------------------------------------------|
|ExtentId   |`guid`    |Identificador único de la partición de datos generada por el comando.|
|ItemLoaded |`string`  |Uno o varios artefactos de almacenamiento relacionados con este registro.             |
|Duration   |`timespan`|Cuánto tiempo tardó en realizarse la ingesta.                                     |
|HasErrors  |`bool`    |Indica si este registro representa o no un error de ingesta.                |
|OperationId|`guid`    |IDENTIFICADOR único que representa la operación. Se puede usar con el `.show operation` comando.|

**Comentarios:**

Este comando no modifica el esquema de la tabla que se va a ingerir en.
Si es necesario, los datos se "convierten" en este esquema durante la ingesta, no al revés (las columnas adicionales se omiten y las columnas que faltan se tratan como valores NULL).

**Ejemplos**

En el ejemplo siguiente se indica al motor que lea dos blobs de Azure Blob Storage como archivos CSV y que recopile su contenido en la tabla `T` . `...`Representa una Azure Storage firma de acceso compartido (SAS) que proporciona acceso de lectura a cada BLOB. Tenga en cuenta también el uso de cadenas ofuscadas ( `h` delante de los valores de cadena) para asegurarse de que nunca se registra la SAS.

```kusto
.ingest into table T (
    h'https://contoso.blob.core.windows.net/container/file1.csv?...',
    h'https://contoso.blob.core.windows.net/container/file2.csv?...'
)
```

El ejemplo siguiente es para la ingesta de datos de Azure Data Lake Storage Gen 2 (ADLSv2). Las credenciales usadas aquí ( `...` ) son las credenciales de la cuenta de almacenamiento (clave compartida) y usamos la ofuscación de cadenas solo para la parte secreta de la cadena de conexión.

```kusto
.ingest into table T (
  'abfss://myfilesystem@contoso.dfs.core.windows.net/path/to/file1.csv;'
    h'...'
)
```

En el ejemplo siguiente se ingesta un único archivo de Azure Data Lake Storage (ADLS).
Usa las credenciales del usuario para obtener acceso a ADLS (por lo que no hay necesidad de tratar el URI de almacenamiento como un secreto). También se muestra cómo especificar las propiedades de ingesta.

```kusto
.ingest into table T ('adl://contoso.azuredatalakestore.net/Path/To/File/file1.ext;impersonate')
  with (format='csv')
```

