---
title: 'Asignación de ingesta de .alter: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe la asignación de ingesta .alter en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 5343d55fadafce552c5d837e5eb50763ccf45a4c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522413"
---
# <a name="alter-ingestion-mapping"></a>.alter ingestion mapping

Altera una asignación de ingesta existente asociada a una tabla específica y a un formato específico (reemplazar la asignación completa).

**Sintaxis**

`.alter``table` *TableName* `ingestion` *MappingKind* `mapping` *MappingName* *MappingFormattedAsJson*

> [!NOTE]
> * Se puede hacer referencia a esta asignación mediante comandos de ingesta, en lugar de especificar la asignación completa como parte del comando.
> * Los valores válidos `CSV`para `JSON` `avro` _MappingKind_ `orc`son: , , , `parquet`, y .

**Ejemplo** 
 
```
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

| NOMBRE     | Clase | Asignación                                                                                                                                                                          |
|----------|------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| mapeo1 | CSV  | ['"Name":"rownumber","DataType":"int","CsvDataType":null,"Ordinal":0,"ConstValue":null-,"Name":"rowguid","DataType":"string","CsvDataType":null,"Ordinal":1,"ConstValue":null-] |