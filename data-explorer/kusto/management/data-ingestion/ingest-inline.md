---
title: El comando en línea .ingest (push) - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe el comando en línea .ingest (push) en el Explorador de azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 47da2df6ff974afb5e91224ade695dc0863b6817
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521376"
---
# <a name="the-ingest-inline-command-push"></a>El comando .ingest en línea (push)

Este comando ingesta los datos en una tabla "empujando" los datos incrustados en línea en el propio texto del comando.

> [!NOTE]
> El propósito principal de este comando es para fines de prueba ad hoc manuales.
> Para el uso de producción se recomienda utilizar otros métodos de ingesta más adecuados para la entrega masiva de grandes cantidades de datos, como [la ingesta del almacenamiento.](./ingest-from-storage.md)

**Sintaxis**

`.ingest``inline` `(` *TableName* `with` `=` *IngestionPropertyValue* `,` *IngestionPropertyName* TableName [ IngestionPropertyName IngestionPropertyValue [ ...] `into` `table` `)`] `<|` *Datos*



**Argumentos**

* *TableName* es el nombre de la tabla en la que se ingentan los datos.
  El nombre de la tabla siempre es relativo a la base de datos en contexto y su esquema es el esquema que se asumirá para los datos si no se proporciona ningún objeto de asignación de esquema.

* *Los datos* son el contenido de los datos que se va a ingerir. A menos que las propiedades de ingesta modifiquen lo contrario, este contenido se analiza como CSV.
  Tenga en cuenta que, a diferencia de *Data* la mayoría de los comandos y consultas de control, el texto de la parte Datos `//` del comando no tiene que seguir las convenciones sintácticas del lenguaje (por ejemplo, los caracteres de espacio en blanco son importantes, la combinación no se trata como un comentario, etc.)

* *IngestionPropertyName*, *IngestionPropertyValue*: cualquier número de propiedades de [ingesta](https://docs.microsoft.com/azure/data-explorer/ingestion-properties) que afecten al proceso de ingesta.

**Resultados**

El resultado del comando es una tabla con tantos registros como particiones de datos ("extensiones") generadas por el comando.
Si no se han generado particiones de datos, se devuelve un único registro con un identificador de extensión vacío (de valor cero).

|Nombre       |Tipo      |Descripción                                                                |
|-----------|----------|---------------------------------------------------------------------------|
|ExtentId   |`guid`    |Identificador único de la partición de datos generada por el comando.|

**Ejemplos**

El siguiente comando ingesta datos`Purchases`en una tabla `SKU` ( `string`) `Quantity` con dos `long`columnas, (de tipo ) y (de tipo ).

```kusto
.ingest inline into table Purchases <|
Shoes,1000
Wide Shoes,50
"Coats, black",20
"Coats with ""quotes""",5
```



<!--
It is possible to generate inline ingests commands using the Kusto.Data client library. (Note that compression does allow one to embed newlines in quoted fields) 

    Kusto.Data.Common.CslCommandGenerator.GenerateTableIngestPushCommand(tableName, compressed: true, csvData: csvStream);

-->