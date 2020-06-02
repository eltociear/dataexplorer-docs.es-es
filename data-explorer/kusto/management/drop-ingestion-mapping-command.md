---
title: '. Drop ingesta mapping: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la asignación de ingesta de eliminación en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: 7454bd86a6ca2a835dc0515a9c8901a444259f12
ms.sourcegitcommit: 41cd88acc1fd79f320a8fe8012583d4c8522db78
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/02/2020
ms.locfileid: "84294532"
---
# <a name="drop-ingestion-mapping"></a>.drop ingestion mapping

Quita la asignación de ingesta de la base de datos.
 
`.drop``table` *TableName* `ingestion` *MappingKind* `mapping` *MappingName*   

**Ejemplo** 

```kusto
.drop table MyTable ingestion CSV mapping "Mapping1" 

.drop table MyTable ingestion json mapping "Mapping1" 
```
