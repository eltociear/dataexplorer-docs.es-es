---
title: 'Asignaciones de ingesta de .show: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describen las asignaciones de ingesta de .show en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 91990fe40664ae89d69357812b0def2c7288eb7d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81519829"
---
# <a name="show-ingestion-mappings"></a>.show asignaciones de ingesta

Mostrar las asignaciones de ingesta (todas o la especificada por nombre).

* `.show``table` *TableName* `ingestion` *MappingKind*  `mappings`

* `.show``table` *TableName* `ingestion` *MappingKind* `mapping` *MappingName*   

Mostrar todas las asignaciones de ingesta de todos los tipos de asignación:

* `.show``table` *TableName*`ingestion`  `mapping`
 
**Ejemplo** 
 
```
.show table MyTable ingestion csv mapping "Mapping1" 

.show table MyTable ingestion csv mappings 

.show table MyTable ingestion mappings 
```

**Salida del ejemplo**

| NOMBRE     | Clase | Asignación     |
|----------|------|-------------|
| mapeo1 | CSV  | ['"Name":"rownumber","DataType":"int","CsvDataType":null,"Ordinal":0,"ConstValue":null-,"Name":"rowguid","DataType":"string","CsvDataType":null,"Ordinal":1,"ConstValue":null-] |