---
title: 'operador de datos externos: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador externaldata en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 0a4a151d1fd052e27e4daf76539ef6dbd9d94c83
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515477"
---
# <a name="externaldata-operator"></a>Operador externaldata

El `externaldata` operador devuelve una tabla cuyo esquema se define en la propia consulta y cuyos datos se leen desde un artefacto de almacenamiento externo (por ejemplo, un blob en Azure Blob Storage).

> [!NOTE]
> Este operador no tiene una entrada de canalización.

**Sintaxis**

`externaldata``(` *ColumnName* `:` *ColumnType* [`,` ...] `)` `]` `with` `(` `=` *Value1* `,` *Prop1* *StorageConnectionString* StorageConnectionString [ Prop1 Value1 [ ...] `[` `)`]

**Argumentos**

* *ColumnName*, *ColumnType*: Defina el esquema de la tabla.
  La sintaxis es la misma que la sintaxis utilizada al definir una tabla en [la tabla .create.](../management/create-table-command.md)

* *StorageConnectionString*: la cadena de conexión de [almacenamiento](../api/connection-strings/storage.md) describe el artefacto de almacenamiento que contiene los datos que se devolverán.

* *Prop1*, *Value1*, ...: Propiedades adicionales que describen cómo interpretar los datos recuperados del almacenamiento, tal como se enumeran en Propiedades de [ingesta](../management/data-ingestion/index.md).
    * Propiedades admitidas `format` `ignoreFirstRecord`actualmente: y .
    * Formatos de datos compatibles: cualquiera de los `csv` `tsv`formatos de datos de [ingesta](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats) son compatibles, incluidos , , `json`, `parquet`. `avro`.

**Devuelve**

El `externaldata` operador devuelve una tabla de datos del esquema especificado cuyos datos se analizaron desde el artefacto de almacenamiento especificado indicado por la cadena de conexión de almacenamiento.

**Ejemplo**

En el ejemplo siguiente se muestra cómo `UserID` buscar todos los registros de una tabla cuya columna cae en un conjunto conocido de ID, mantenidos (uno por línea) en un blob externo.
Dado que la consulta hace referencia indirectamente al conjunto, puede ser muy grande.

```
Users
| where UserID in ((externaldata (UserID:string) [
    @"https://storageaccount.blob.core.windows.net/storagecontainer/users.txt"
      h@"?...SAS..." // Secret token needed to access the blob
    ]))
| ...
```