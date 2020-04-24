---
title: 'Exportar datos al almacenamiento: Explorador de Azure Data Explorer . Microsoft Docs'
description: En este artículo se describe Exportar datos al almacenamiento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/12/2020
ms.openlocfilehash: 12800955d1680a280aa4772db86d8b71e44e7089
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521580"
---
# <a name="export-data-to-storage"></a>Exportar datos al almacenamiento

Ejecuta una consulta y escribe el primer conjunto de resultados en un almacenamiento externo, especificado por una cadena de conexión de [almacenamiento.](../../api/connection-strings/storage.md)

**Sintaxis**

`.export`[`async`]`compressed` `to` [ ] *OutputDataFormat* 
 `(` *StorageConnectionString* [`,` ...] `)` `with` [ `(` *PropertyName* `=` *PropertyValue* [`,` ...] `)`] `<|` *Consulta*

**Argumentos**

* `async`: Si se especifica, indica que el comando se ejecuta en modo asincrónico.
  Consulte a continuación para obtener más detalles sobre el comportamiento en este modo.

* `compressed`: si se especifica, los artefactos de almacenamiento de salida se comprimen como `.gz` archivos. Consulte `compressionType` para comprimir archivos de parquet como ágiles. 

* *OutputDataFormat*: Indica el formato de datos de los artefactos de almacenamiento escritos por el comando. Los valores `csv`admitidos son: , `tsv`, `json`, y `parquet`.

* *StorageConnectionString*: especifica una o varias cadenas de conexión de [almacenamiento](../../api/connection-strings/storage.md) que indican en qué almacenamiento escribir los datos. (Se puede especificar más de una cadena de conexión de almacenamiento para escrituras escalables.) Cada una de estas cadenas de conexión debe indicar las credenciales que se deben usar al escribir en el almacenamiento.
  Por ejemplo, al escribir en Azure Blob Storage, las credenciales pueden ser la clave de cuenta de almacenamiento o una clave de acceso compartido (SAS) con permisos para leer, escribir y enumerar blobs.

> [!NOTE]
> Se recomienda encarecidamente exportar datos al almacenamiento que se encuentra en la misma región que el propio clúster de Kusto. Esto incluye los datos que se exportan para que se puedan transferir a otro servicio en la nube en otras regiones. Las escrituras deben realizarse localmente, mientras que las lecturas pueden ocurrir de forma remota.

* *PropertyName*/*PropertyValue*: Cero o más propiedades de exportación opcionales:

|Propiedad        |Tipo    |Descripción                                                                                                                |
|----------------|--------|---------------------------------------------------------------------------------------------------------------------------|
|`sizeLimit`     |`long`  |El límite de tamaño en bytes de un único artefacto de almacenamiento que se está escribiendo (antes de la compresión). El rango permitido es de 100 MB (predeterminado) a 1 GB.|
|`includeHeaders`|`string`|`csv` / Para `tsv` la salida, controla la generación de encabezados de columna. Puede ser `none` uno de (predeterminado; no `all` se emiten líneas de encabezado), (emitir una línea de encabezado en cada artefacto de almacenamiento) o `firstFile` (emitir una línea de encabezado solo en el primer artefacto de almacenamiento).|
|`fileExtension` |`string`|Indica la parte "extensión" del artefacto de `.csv` `.tsv`almacenamiento (por ejemplo, o ). Si se utiliza `.gz` la compresión, se añadirá también.|
|`namePrefix`    |`string`|Indica un prefijo que se va a agregar a cada nombre de artefacto de almacenamiento generado. Se utilizará un prefijo aleatorio si no se especifica.       |
|`encoding`      |`string`|Indica cómo codificar el texto: `UTF8NoBOM` (predeterminado) o `UTF8BOM`. |
|`compressionType`|`string`|Indica el tipo de compresión que se va a utilizar. Los valores posibles son `gzip` o `snappy`. El valor predeterminado es `gzip`. `snappy`puede (opcionalmente) ser `parquet` utilizado para el formato. |
|`distribution`   |`string`  |Sugerencia de`single` `per_node`distribución ( , , `per_shard`). Si value `single`es igual a , un único subproceso escribirá en el almacenamiento. De lo contrario, export escribirá desde todos los nodos que ejecutan la consulta en paralelo. Consulte [evaluar operador de plugin](../../query/evaluateoperator.md). Su valor predeterminado es `per_shard`.
|`distributed`   |`bool`  |Deshabilite/habilite la exportación distribuida. Establecer en false `single` es equivalente a la sugerencia de distribución. El valor predeterminado es true.
|`persistDetails`|`bool`  |Indica que el comando debe conservar `async` sus resultados (consulte la marca). El valor `true` predeterminado es en ejecuciones asincrónicas, pero se puede desactivar si el autor de la llamada no requiere los resultados). El valor `false` predeterminado es en las ejecuciones sincrónicas, pero también se puede activar en ellas. |
|`parquetRowGroupSize`|`int`  |Relevante sólo cuando el formato de datos es parquet. Controla el tamaño del grupo de filas en los archivos exportados. El tamaño predeterminado del grupo de filas es 100000 registros.|

**Resultados**

Los comandos devuelven una tabla que describe los artefactos de almacenamiento generados.
Cada registro describe un único artefacto e incluye la ruta de acceso de almacenamiento al artefacto y cuántos registros de datos contiene.

|Path|NumRecords|
|---|---|
|http://storage1.blob.core.windows.net/containerName/export_1_d08afcae2f044c1092b279412dcb571b.csv|10|
|http://storage1.blob.core.windows.net/containerName/export_2_454c0f1359e24795b6529da8a0101330.csv|15|

**Modo asincrónico**

Si `async` se especifica el indicador, el comando se ejecuta en modo asincrónico.
En este modo, el comando vuelve inmediatamente con un identificador de operación y la exportación de datos continúa en segundo plano hasta su finalización. El ID de operación devuelto por el comando se puede utilizar para realizar un seguimiento de su progreso y, en última instancia, sus resultados a través de los siguientes comandos:

* [.show operaciones](../operations.md#show-operations): Seguimiento del progreso.
* [Detalles de la operación .show](../operations.md#show-operation-details): Obtener resultados de finalización.

Por ejemplo, después de una finalización correcta, puede recuperar los resultados mediante:

```kusto
.show operation f008dc1e-2710-47d8-8d34-0d562f5f8615 details
```

**Ejemplos** 

En este ejemplo, Kusto ejecuta la consulta y, a continuación, exporta el primer conjunto de registros generado por la consulta a uno o varios blobs CSV comprimidos.
Las etiquetas de nombre de columna se agregan como la primera fila para cada blob.

```kusto 
.export
  async compressed
  to csv (
    h@"https://storage1.blob.core.windows.net/containerName;secretKey",
    h@"https://storage1.blob.core.windows.net/containerName2;secretKey"
  ) with (
    sizeLimit=100000,
    namePrefix=export,
    includeHeaders=all,
    encoding =UTF8NoBOM
  )
  <| myLogs | where id == "moshe" | limit 10000
```

**Problemas conocidos**

*Errores de almacenamiento durante el comando de exportación*

De forma predeterminada, el comando export se distribuye de forma que todas las [extensiones](../extents-overview.md) que contienen datos para exportar escriben en el almacenamiento simultáneamente. En las exportaciones grandes, cuando el número de estas extensiones es alto, esto puede dar lugar a una carga elevada en el almacenamiento que da lugar a una limitación del almacenamiento de información o a errores de almacenamiento transitorios. En tales casos, se recomienda intentar aumentar el número de cuentas de almacenamiento proporcionadas al comando de exportación (la carga `per_node` se distribuirá entre las cuentas) o reducir la simultaneidad estableciendo la sugerencia de distribución en (consulte las propiedades del comando). También es posible deshabilitar completamente la distribución, pero esto puede afectar significativamente al rendimiento del comando.
 