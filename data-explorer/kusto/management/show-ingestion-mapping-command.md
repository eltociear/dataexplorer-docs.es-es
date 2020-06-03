---
title: '. Mostrar las asignaciones de ingesta: Azure Explorador de datos'
description: En este artículo se describe. Mostrar las asignaciones de ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 3c19426410046d7ff2357b4967333db8b039d9e6
ms.sourcegitcommit: f7101c6b41ec250d05f4cb6092e2939958b37b40
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2020
ms.locfileid: "84329000"
---
# <a name="show-ingestion-mappings"></a>. Mostrar asignaciones de ingesta

Muestra las asignaciones de ingesta (todas o las especificadas por nombre).

* `.show``table` *TableName* `ingestion` *MappingKind*  `mappings`

* `.show``table` *TableName* `ingestion` *MappingKind* `mapping` *MappingName*   

Mostrar todas las asignaciones de ingesta de todos los tipos de asignación:

* `.show``table` *TableName*`ingestion`  `mapping`
 
**Ejemplo** 
 
```kusto
.show table MyTable ingestion csv mapping "Mapping1" 

.show table MyTable ingestion csv mappings 

.show table MyTable ingestion mappings 
```

**Salida de ejemplo**

| Nombre     | Tipo | Asignación     |
|----------|------|-------------|
| mapping1 | CSV  | `[{"Name":"rownumber","DataType":"int","CsvDataType":null,"Ordinal":0,"ConstValue":null},{"Name":"rowguid","DataType":"string","CsvDataType":null,"Ordinal":1,"ConstValue":null}]` |
