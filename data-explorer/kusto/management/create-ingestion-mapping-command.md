---
title: 'Asignación de ingesta de .create: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe la asignación de ingesta de .create en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 10e656b074516ad8b0018e627d9904251aebbf10
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744495"
---
# <a name="create-ingestion-mapping"></a>.create ingestion mapping

Crea una asignación de ingesta asociada a una tabla específica y a un formato específico.

**Sintaxis**

`.create``table` *TableName* `ingestion` *MappingKind* `mapping` *MappingName* *MappingFormattedAsJson*

> [!NOTE]
> * Una vez creada, se puede hacer referencia a la asignación por su nombre en los comandos de ingesta, en lugar de especificar la asignación completa como parte del comando.
> * Los valores válidos `CSV`para `JSON` `avro` _MappingKind_ son: , , , `parquet`, y`orc`
> * Si ya existe una asignación con el mismo nombre para la tabla:
>    * `.create`fallará
>    * `.create-or-alter`alterará la cartografía existente
 
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

| NOMBRE     | Clase | Asignación                                                                                                                                                                          |
|----------|------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| mapeo1 | CSV  | ['"Name":"rownumber","DataType":"int","CsvDataType":null,"Ordinal":0,"ConstValue":null-,"Name":"rowguid","DataType":"string","CsvDataType":null,"Ordinal":1,"ConstValue":null-] |
