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
ms.openlocfilehash: e35245cf767e3cf82ab61d5ce0704015d996cd7c
ms.sourcegitcommit: 283cce0e7635a2d8ca77543f297a3345a5201395
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2020
ms.locfileid: "84011387"
---
# <a name="externaldata-operator"></a>Operador externaldata

El `externaldata` operador devuelve una tabla cuyo esquema se define en la propia consulta y cuyos datos se leen de un artefacto de almacenamiento externo (como un BLOB en Azure BLOB Storage).

> [!NOTE]
> Este operador no tiene una entrada de canalización.

**Sintaxis**

`externaldata``(` *ColumnName* `:` *ColumnType* [ `,` ...] `)` `[` *StorageConnectionString* `]` [ `with` `(` *Prop1* `=` *Value1* [ `,` ...] `)` ]

**Argumentos**

* *ColumnName*, *ColumnType*: define el esquema de la tabla.
  La sintaxis es la misma que la que se usa al definir una tabla en [. CREATE TABLE](../management/create-table-command.md).

* *StorageConnectionString*: la [cadena de conexión de almacenamiento](../api/connection-strings/storage.md) describe el artefacto de almacenamiento que contiene los datos que se van a devolver.

* *Prop1*, *Value1*,...: propiedades adicionales que describen cómo interpretar los datos recuperados del almacenamiento, como se muestra en [propiedades de ingesta](../../ingestion-properties.md).
    * Propiedades admitidas actualmente: `format` y `ignoreFirstRecord` .
    * Formatos de datos admitidos: se admite cualquiera de los [formatos de datos de ingesta](../../ingestion-supported-formats.md) , incluidos `csv` ,, `tsv` `json` , `parquet` , `avro` .

**Devuelve**

El `externaldata` operador devuelve una tabla de datos del esquema dado cuyos datos se analizaron a partir del artefacto de almacenamiento especificado indicado por la cadena de conexión de almacenamiento.

**Ejemplo**

En el ejemplo siguiente se muestra cómo buscar todos los registros de una tabla cuya `UserID` columna cae en un conjunto conocido de identificadores, mantenidos (uno por línea) en un BLOB externo.
Dado que la consulta hace referencia indirectamente al conjunto, puede ser muy grande.

```
Users
| where UserID in ((externaldata (UserID:string) [
    @"https://storageaccount.blob.core.windows.net/storagecontainer/users.txt"
      h@"?...SAS..." // Secret token needed to access the blob
    ]))
| ...
```