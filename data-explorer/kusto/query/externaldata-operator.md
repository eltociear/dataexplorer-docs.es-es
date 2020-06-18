---
title: 'operador externaldata: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe el operador externaldata en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: f8878a3c4589dca3cfacf935a787e8c754ab3ede
ms.sourcegitcommit: a8575e80c65eab2a2118842e59f62aee0ff0e416
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2020
ms.locfileid: "84942682"
---
# <a name="externaldata-operator"></a>Operador externaldata

El `externaldata` operador devuelve una tabla cuyo esquema se define en la propia consulta y cuyos datos se leen de un artefacto de almacenamiento externo (como un BLOB en Azure BLOB Storage).

**Sintaxis**

`externaldata``(` *ColumnName* `:` *ColumnType* [ `,` ...] `)` `[` *StorageConnectionString* `]` [ `with` `(` *Prop1* `=` *Value1* [ `,` ...] `)` ]

**Argumentos**

* *ColumnName*, *ColumnType*: define el esquema de la tabla.
  La sintaxis es la misma que la que se usa al definir una tabla en [. CREATE TABLE](../management/create-table-command.md).

* *StorageConnectionString*: la [cadena de conexión de almacenamiento](../api/connection-strings/storage.md) describe el artefacto de almacenamiento que contiene los datos que se van a devolver.

* *Prop1*, *Value1*,...: propiedades adicionales que describen cómo interpretar los datos recuperados del almacenamiento, como se muestra en [propiedades de ingesta](../../ingestion-properties.md).
    * Propiedades admitidas actualmente: `format` y `ignoreFirstRecord` .
    * Formatos de datos admitidos: se admite cualquiera de los [formatos de datos de ingesta](../../ingestion-supported-formats.md) , incluidos `CSV` ,, `TSV` `JSON` , `Parquet` , `Avro` .

> [!NOTE]
> Este operador no tiene una entrada de canalización.

**Devuelve**

El `externaldata` operador devuelve una tabla de datos del esquema dado cuyos datos se analizaron a partir del artefacto de almacenamiento especificado indicado por la cadena de conexión de almacenamiento.

**Ejemplos**

En el ejemplo siguiente se muestra cómo buscar todos los registros de una tabla cuya `UserID` columna cae en un conjunto conocido de identificadores, mantenidos (uno por línea) en un BLOB externo.
Dado que la consulta hace referencia indirectamente al conjunto, puede ser grande.

```kusto
Users
| where UserID in ((externaldata (UserID:string) [
    @"https://storageaccount.blob.core.windows.net/storagecontainer/users.txt"
      h@"?...SAS..." // Secret token needed to access the blob
    ]))
| ...
```

En el ejemplo siguiente se consultan varios archivos de datos almacenados en el almacenamiento externo.

```kusto
externaldata(Timestamp:datetime, ProductId:string, ProductDescription:string)
[
  h@"https://mycompanystorage.blob.core.windows.net/archivedproducts/2019/01/01/part-00000-7e967c99-cf2b-4dbb-8c53-ce388389470d.csv.gz?...SAS...",
  h@"https://mycompanystorage.blob.core.windows.net/archivedproducts/2019/01/02/part-00000-ba356fa4-f85f-430a-8b5a-afd64f128ca4.csv.gz?...SAS...",
  h@"https://mycompanystorage.blob.core.windows.net/archivedproducts/2019/01/03/part-00000-acb644dc-2fc6-467c-ab80-d1590b23fc31.csv.gz?...SAS..."
]
with(format="csv")
| summarize count() by ProductId
```

El ejemplo anterior se puede considerar una manera rápida de consultar varios archivos de datos sin definir una [tabla externa](schema-entities/externaltables.md). 

>[!NOTE]
>El operador no reconoce las particiones de datos `externaldata()` .
