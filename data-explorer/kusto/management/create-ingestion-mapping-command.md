---
title: '. crear asignación de ingesta: Azure Explorador de datos'
description: En este artículo se describe. creación de una asignación de ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 3855d56ad31bbf98a6a075feb44a598b3bdbf52a
ms.sourcegitcommit: f7101c6b41ec250d05f4cb6092e2939958b37b40
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2020
ms.locfileid: "84329068"
---
# <a name="create-ingestion-mapping"></a>.create ingestion mapping

Crea una asignación de ingesta que está asociada a una tabla específica y a un formato específico.

**Sintaxis**

`.create``table` *TableName* `ingestion` *MappingKind* `mapping` *MappingName* *MappingFormattedAsJson*

> [!NOTE]
> * Una vez creada, se puede hacer referencia a la asignación por su nombre en los comandos de ingesta, en lugar de especificar la asignación completa como parte del comando.
> * Los valores válidos para _MappingKind_ son: `CSV` , `JSON` , `avro` , `parquet` y`orc`
> * Si ya existe una asignación con el mismo nombre para la tabla:
>    * `.create`producirá un error
>    * `.create-or-alter`modificará la asignación existente
 
**Ejemplo** 
 
```kusto
.create table MyTable ingestion csv mapping "Mapping1"
'['
'   { "column" : "rownumber", "DataType":"int", "Properties":{"Ordinal":"0"}},'
'   { "column" : "rowguid", "DataType":"string", "Properties":{"Ordinal":"1"}}'
']'

.create-or-alter table MyTable ingestion json mapping "Mapping1"
'['
'    { "column" : "rownumber", "datatype" : "int", "Properties":{"Path":"$.rownumber"}},'
'    { "column" : "rowguid", "Properties":{"Path":"$.rowguid"}}'
']'
```

**Salida del ejemplo**

| Nombre     | Tipo | Asignación                                                                                                                                                                          |
|----------|------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| mapping1 | CSV  | `[{"Name":"rownumber","DataType":"int","CsvDataType":null,"Ordinal":0,"ConstValue":null},{"Name":"rowguid","DataType":"string","CsvDataType":null,"Ordinal":1,"ConstValue":null}]` |

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre la asignación de ingesta, vea [asignaciones de datos](mappings.md).
