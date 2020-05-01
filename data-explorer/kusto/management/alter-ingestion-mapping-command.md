---
title: '. modificar asignación de ingesta: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la asignación de ingesta de cambios en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 2f43039ff3935edbb6e92627d2f96b1c411e1ffa
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617823"
---
# <a name="alter-ingestion-mapping"></a>.alter ingestion mapping

Modifica una asignación de ingesta existente que está asociada a una tabla específica y a un formato específico (reemplazo de asignación completo).

**Sintaxis**

`.alter``table` *TableName* TableName `ingestion` *MappingKind* MappingKind `mapping` *MappingName* *MappingFormattedAsJson*

> [!NOTE]
> * Se puede hacer referencia a esta asignación por su nombre mediante comandos de ingesta, en lugar de especificar la asignación completa como parte del comando.
> * Los valores válidos para _MappingKind_ son `JSON`: `avro` `CSV`, `parquet`,, `orc`y.

**Ejemplo** 
 
```kusto
.alter table MyTable ingestion csv mapping "Mapping1"
'['
'   { "column" : "rownumber", "DataType":"int", "Properties":{"Ordinal":"0"}},'
'   { "column" : "rowguid", "DataType":"string", "Properties":{"Ordinal":"1"} }'
']'

.alter table MyTable ingestion json mapping "Mapping1"
'['
'   { "column" : "rownumber", "Properties":{"Path":"$.rownumber"}},'
'   { "column" : "rowguid", "Properties":{"Path":"$.rowguid"}}'
']'
```

**Salida del ejemplo**

| Nombre     | Clase | Asignación                                                                                                                                                                          |
|----------|------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| mapping1 | CSV  | [{"Name": "RowNumber", "DataType": "int", "CsvDataType": null, "ordinal": 0, "ConstValue": null}, {"Name": "ROWGUID", "DataType": "String", "CsvDataType": null, "ordinal": 1, "ConstValue": null}] |
