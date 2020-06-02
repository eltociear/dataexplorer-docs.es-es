---
title: 'Comando insertado de ingesta (inserción): Azure Explorador de datos'
description: En este artículo se describe el comando en línea. ingesta (inserción).
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 35098d2605a637832fd513da62a956382183a47c
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84294549"
---
# <a name="ingest-inline-command-push"></a>. ingesta (comando insertado)

Este comando ingeri los datos en una tabla mediante la "inserción" de los datos insertados en línea, en el propio texto de comando.

> [!NOTE]
> Este comando se usa para las pruebas ad hoc manuales.
> Para su uso en producción, se recomienda usar otros métodos de ingesta que son mejores para la entrega masiva de grandes cantidades de datos, como la [ingesta del almacenamiento](./ingest-from-storage.md).

**Sintaxis**

`.ingest``inline` `into` `table` *TableName* [ `with` `(` *IngestionPropertyName* `=` *IngestionPropertyValue* [ `,` ...] `)` ] `<|` *Datos* de

**Argumentos**

* *TableName* es el nombre de la tabla en la que se van a ingerir datos.
  El nombre siempre está relacionado con la base de datos en contexto.
  El esquema de tabla es el esquema que se asumirá para los datos si no se proporciona ningún objeto de asignación de esquema.

* Los *datos* son el contenido de los datos para ingerir. A menos que las propiedades de ingesta modifiquen lo contrario, este contenido se analiza como CSV.
 
 > [!NOTE]
 > A diferencia de la mayoría de los comandos de control y las consultas, el texto de la parte de *datos* del comando no tiene que seguir las convenciones sintácticas del lenguaje. Por ejemplo, los caracteres de espacio en blanco son importantes o la `//` combinación no se trata como un comentario.

* *IngestionPropertyName*, *IngestionPropertyValue*: cualquier número de [propiedades de ingesta](../../../ingestion-properties.md) que afecten al proceso de ingesta.

**Resultados**

El resultado es una tabla con tantos registros como el número de particiones de datos generadas ("extensiones").
Si no se generan particiones de datos, se devuelve un único registro con un identificador de extensión vacío (valor cero).

|Nombre       |Tipo      |Descripción                                                               |
|-----------|----------|--------------------------------------------------------------------------|
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
You can generate inline ingests commands using the Kusto.Data client library. 
Compression lets you embed new lines in quoted fields.

    Kusto.Data.Common.CslCommandGenerator.GenerateTableIngestPushCommand(tableName, compressed: true, csvData: csvStream);

-->
