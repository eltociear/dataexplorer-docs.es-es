---
title: 'Comando en línea de ingesta (inserción): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe el comando en línea de ingesta (inserción) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 2b1766b295fd348639d8d91c8308a3ed0a35a3dc
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373409"
---
# <a name="the-ingest-inline-command-push"></a>Comando en línea. ingesta (inserción)

Este comando ingeri los datos en una tabla mediante la "inserción" de los datos insertados en el propio texto de comando.

> [!NOTE]
> Este es el propósito principal de este comando es para la realización de pruebas ad hoc manuales.
> En el caso del uso de producción, se recomienda usar otros métodos de ingesta más adecuados para la entrega masiva de grandes cantidades de datos, como la [ingesta del almacenamiento](./ingest-from-storage.md).

**Sintaxis**

`.ingest``inline` `into` `table` *TableName* [ `with` `(` *IngestionPropertyName* `=` *IngestionPropertyValue* [ `,` ...] `)` ] `<|` *Datos* de



**Argumentos**

* *TableName* es el nombre de la tabla en la que se van a ingerir datos.
  El nombre de la tabla siempre es relativo a la base de datos en contexto, y su esquema es el esquema que se asumirá para los datos si no se proporciona ningún objeto de asignación de esquema.

* Los *datos* son el contenido de los datos para ingerir. A menos que las propiedades de ingesta modifiquen lo contrario, este contenido se analiza como CSV.
  Tenga en cuenta que, a diferencia de la mayoría de los comandos de control y las consultas, el texto de la parte de *datos* del comando no tiene que seguir las convenciones sintácticas del lenguaje (por ejemplo, los caracteres de espacio en blanco son importantes, la `//` combinación no se trata como comentario, etc.).

* *IngestionPropertyName*, *IngestionPropertyValue*: cualquier número de [propiedades de ingesta](../../../ingestion-properties.md) que afecten al proceso de ingesta.

**Resultados**

El resultado del comando es una tabla con tantos registros como particiones de datos ("extensiones") generados por el comando.
Si no se han generado particiones de datos, se devuelve un único registro con un identificador de extensión vacío (de valor cero).

|Nombre       |Tipo      |Descripción                                                                |
|-----------|----------|---------------------------------------------------------------------------|
|ExtentId   |`guid`    |Identificador único de la partición de datos generada por el comando.|

**Ejemplos**

El comando siguiente ingeri datos en una tabla ( `Purchases` ) con dos columnas, `SKU` (de tipo `string` ) y `Quantity` (de tipo `long` ).

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