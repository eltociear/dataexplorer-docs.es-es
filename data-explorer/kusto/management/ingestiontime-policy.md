---
title: 'Administración de directivas de Kusto IngestionTime: Azure Explorador de datos'
description: En este artículo se describe la Directiva IngestionTime en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 907c6ddf84d772f800fce45d3c1245bbd11b0c85
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82616463"
---
# <a name="ingestiontime-policy"></a>Directiva de IngestionTime

La Directiva IngestionTime es un conjunto de directivas opcional en las tablas (está habilitada de forma predeterminada).
proporciona el tiempo aproximado de ingesta de los registros en una tabla.

Se puede tener acceso al valor de tiempo de ingesta en `ingestion_time()` el momento de la consulta mediante la función.

```kusto
T | extend ingestionTime = ingestion_time()
```

Para habilitar o deshabilitar la Directiva:
```kusto
.alter table table_name policy ingestiontime true
```

Para habilitar o deshabilitar la Directiva de varias tablas:
```kusto
.alter tables (table_name [, ...]) policy ingestiontime true
```

Para ver la Directiva:
```kusto
.show table table_name policy ingestiontime  

.show table * policy ingestiontime  
```

Para eliminar la Directiva (es decir, deshabilitar):
```kusto
.delete table table_name policy ingestiontime  
```