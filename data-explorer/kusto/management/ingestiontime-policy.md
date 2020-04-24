---
title: Directiva IngestionTime- Explorador de Azure Data Explorer ( IngestionTime) Microsoft Docs
description: En este artículo se describe la directiva IngestionTime en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: c42da570b961595be1fbcae352fe121d8b6f59ea
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520900"
---
# <a name="ingestiontime-policy"></a>Directiva de IngestionTime

La directiva IngestionTime es un conjunto de políticas opcional en tablas (está habilitada de forma predeterminada).
proporciona el tiempo aproximado de ingestión de los registros en una tabla.

Se puede tener acceso al valor `ingestion_time()` de tiempo de ingesta en el momento de la consulta mediante la función.

```kusto
T | extend ingestionTime = ingestion_time()
```

Para activar/desactivar la directiva:
```kusto
.alter table table_name policy ingestiontime true
```

Para habilitar/deshabilitar la directiva de varias tablas:
```kusto
.alter tables (table_name [, ...]) policy ingestiontime true
```

Para ver la directiva:
```kusto
.show table table_name policy ingestiontime  

.show table * policy ingestiontime  
```

Para eliminar la directiva (igual a deshabilitarla):
```kusto
.delete table table_name policy ingestiontime  
```