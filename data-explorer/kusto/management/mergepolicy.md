---
title: 'Directiva de combinación de extensiones: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la directiva de combinación de extensiones en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 771af22f07a770b0da1196871e9132393524aab9
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520730"
---
# <a name="extents-merge-policy"></a>Política de fusión de extensiones
La directiva de combinación define si y cómo se deben combinar las [extensiones (fragmentos](../management/extents-overview.md) de datos) en el clúster de Kusto.

Hay 2 sabores para `Merge` las operaciones de `Rebuild` combinación: (que reconstruye índices) y (que reings completamente los datos).

Ambos tipos de operación dan como resultado una única extensión que reemplaza las extensiones de origen.

De forma predeterminada, se prefieren las operaciones de reconstrucción y solo si hay alguna extensión restante que no se ajuste a los criterios para volver a generarse, se intentan combinar.  

*Notas:*
- El etiquetado de extensiones mediante etiquetas *diferentes* `drop-by` hará que dichas extensiones no se combinen, incluso si se ha establecido una directiva de combinación (consulte Etiquetado de [extensión](../management/extents-overview.md#extent-tagging)).
- Las extensiones cuya unión de etiquetas supere la longitud de los caracteres 1M no se fusionarán.
- La [directiva Departimiento](./shardingpolicy.md) de la base de datos / tabla también tiene algún efecto en cómo se combinan las extensiones.

La directiva de combinación contiene las siguientes propiedades:

- **RowCountUpperBoundForMerge**:
    - El valor predeterminado es 0.
    - Número máximo de filas permitidos de la extensión combinada.
    - Se aplica a las operaciones de combinación, no a la reconstrucción.  
- **MaxExtentsToMerge**:
    - El valor predeterminado es 100.
    - Número máximo permitido de extensiones que se fusionarán en una sola operación.
    - Se aplica a las operaciones de combinación.
- **LoopPeriod**:
    - El valor predeterminado es 01:00:00 (1 hora).
    - El tiempo máximo de espera entre el inicio de 2 iteraciones consecutivas de operaciones de combinación/reconstrucción (realizadas por el servicio de administración de datos).
    - Se aplica a las operaciones Combinar y Reconstruir.
- **AllowRebuild**:
    - El valor predeterminado es 'true'.
    - Define `Rebuild` si las operaciones están habilitadas (en cuyo caso, se prefieren sobre `Merge` las operaciones).
- **AllowMerge**:
    - El valor predeterminado es 'true'.
    - Define `Merge` si las operaciones están habilitadas (en `Rebuild` cuyo caso, son menos preferidas que las operaciones).
- **MaxRangeInHours**:
    - El valor predeterminado es 8.
    - Diferencia máxima permitida (en horas) entre los tiempos de creación de 2 extensiones diferentes, de modo que todavía se puedan combinar.
    - Las marcas de tiempo son las de la creación de extensiones y no se relacionan con los datos reales contenidos en las extensiones.
    - Se aplica a las operaciones Combinar y Reconstruir.
    - Una práctica recomendada es que este valor se correlacione con la directiva de [retención](./retentionpolicy.md)de la base de datos / table 's *SoftDeletePeriod*, o la directiva de [caché](./cachepolicy.md) *DataHotSpan* (la inferior de las dos), de modo que esté entre el 2 y el 3 % de este último.

**`MaxRangeInHours`Ejemplos:**
|min(SoftDeletePeriod (Política de retención), DataHotSpan (Política de caché))|Rango máximo en horas (directiva de fusión)
|---|---
|7 días (168 horas)| 4
|14 días (336 horas)| 8
|30 días (720 horas)| 18
|60 días (1.440 horas)| 36
|90 días (2.160 horas)| 60
|180 días (4.320 horas)| 120
|365 días (8.760 horas)| 250

> [!WARNING]
> Rara vez se recomienda modificar una política de combinación de extensiones sin consultar primero con el equipo de Kusto.

Cuando se crea una base de datos, se establece con la directiva Merge predeterminada (una directiva con los valores predeterminados mencionados anteriormente), que es, de forma predeterminada, heredada por todas las tablas creadas en la base de datos (a menos que sus directivas se invaliden explícitamente en el nivel de tabla).

Los comandos de control que permiten administrar las directivas de combinación para bases de datos / tablas se pueden encontrar [aquí](../management/merge-policy.md).