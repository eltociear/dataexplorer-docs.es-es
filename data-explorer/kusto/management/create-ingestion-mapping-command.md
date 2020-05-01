---
title: '. crear asignación de ingesta: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe. creación de una asignación de ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 84ab277f5b0d4d1b2e09d31fb7c1254786affe6d
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617738"
---
# <a name="create-ingestion-mapping"></a>.create ingestion mapping

Crea una asignación de ingesta que está asociada a una tabla específica y a un formato específico.

**Sintaxis**

`.create``table` *TableName* TableName `ingestion` *MappingKind* MappingKind `mapping` *MappingName* *MappingFormattedAsJson*

> [!NOTE]
> * Una vez creada, se puede hacer referencia a la asignación por su nombre en los comandos de ingesta, en lugar de especificar la asignación completa como parte del comando.
> * Los valores válidos para _MappingKind_ son `JSON`: `avro` `CSV`, `parquet`,, y`orc`
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

| Nombre     | Clase | Asignación                                                                                                                                                                          |
|----------|------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| mapping1 | CSV  | [{"Name": "RowNumber", "DataType": "int", "CsvDataType": null, "ordinal": 0, "ConstValue": null}, {"Name": "ROWGUID", "DataType": "String", "CsvDataType": null, "ordinal": 1, "ConstValue": null}] |

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información sobre la asignación de ingesta, vea [asignaciones de datos](mappings.md).
