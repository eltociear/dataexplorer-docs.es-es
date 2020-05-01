---
title: '. Mostrar las asignaciones de ingesta: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe. Mostrar las asignaciones de ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 711861a07896b7bdc4cf57bebbf1cdd0e01d064a
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617177"
---
# <a name="show-ingestion-mappings"></a>. Mostrar asignaciones de ingesta

Muestra las asignaciones de ingesta (todas o las especificadas por nombre).

* `.show``table` *TableName* TableName `ingestion` *MappingKind*  `mappings`

* `.show``table` *TableName* TableName `ingestion` *MappingKind* MappingKind`mapping` *MappingName*   

Mostrar todas las asignaciones de ingesta de todos los tipos de asignaciones:

* `.show``table` *TableName*`ingestion`  `mapping`
 
**Ejemplo** 
 
```kusto
.show table MyTable ingestion csv mapping "Mapping1" 

.show table MyTable ingestion csv mappings 

.show table MyTable ingestion mappings 
```

**Salida del ejemplo**

| Nombre     | Clase | Asignación     |
|----------|------|-------------|
| mapping1 | CSV  | [{"Name": "RowNumber", "DataType": "int", "CsvDataType": null, "ordinal": 0, "ConstValue": null}, {"Name": "ROWGUID", "DataType": "String", "CsvDataType": null, "ordinal": 1, "ConstValue": null}] |
