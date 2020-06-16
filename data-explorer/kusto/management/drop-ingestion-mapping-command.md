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
ms.openlocfilehash: 69ab4849d67d7cc7507bab075b0111fbd8ec95e7
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84780616"
---
# <a name="drop-ingestion-mapping"></a>.drop ingestion mapping

Quita la asignación de ingesta de la base de datos.
 
`.drop``table` *TableName* `ingestion` *MappingKind* `mapping` *MappingName*   

**Ejemplo** 

```kusto
.drop table MyTable ingestion csv mapping "Mapping1" 

.drop table MyTable ingestion json mapping "Mapping1" 
```
