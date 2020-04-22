---
title: Asignación de ingestión de .drop- Microsoft Docs
description: En este artículo se describe la asignación de ingesta de .drop en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/04/2020
ms.openlocfilehash: c1434234033acc73de35289c6bc0a90af727babb
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744769"
---
# <a name="drop-ingestion-mapping"></a>.drop ingestion mapping

Quita la asignación de ingesta de la base de datos.
 
`.drop``table` *TableName* `ingestion` *MappingKind* `mapping` *MappingName*   

**Ejemplo** 

```kusto
.drop table MyTable ingestion CSV mapping "Mapping1" 

.drop table MyTable ingestion JSON mappings "Mapping1" 
```
