---
title: 'Directiva de fusión mediante combinación de extensiones: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las extensiones de la Directiva de combinación de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: f0398fbc19842b3cce7fe69c8cb61258d0aeaaa6
ms.sourcegitcommit: a562ce255ac706ca1ca77d272a97b5975235729d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83867042"
---
# <a name="extents-merge-policy"></a>Directiva de combinación de extensiones
La Directiva de combinación define si se deben combinar las [extensiones (particiones de datos)](../management/extents-overview.md) en el clúster de Kusto.

Hay dos tipos de operaciones de combinación: `Merge` (que vuelve a generar los índices) y `Rebuild` (que reingeri los datos por completo).

Ambos tipos de operación dan como resultado una única extensión, que reemplaza las extensiones de origen.

De forma predeterminada, se prefieren las operaciones de recompilación, y solo si hay extensiones restantes que no se ajusten a los criterios para que se vuelvan a generar, se intentarán combinar.  

*Notas:*
- La etiqueta de extensiones mediante etiquetas *diferentes* hará que `drop-by` tales extensiones no se combinen juntas, incluso si se ha establecido una directiva de combinación (consulte [etiquetado de extensiones](../management/extents-overview.md#extent-tagging)).
- Las extensiones cuya unión de etiquetas supera la longitud de 1 millón de caracteres no se combinarán juntas.
- La [Directiva de particionamiento](./shardingpolicy.md) de la base de datos o de la tabla también tiene algún efecto sobre cómo se mezclan las extensiones.

La Directiva de combinación contiene las siguientes propiedades:

- **RowCountUpperBoundForMerge**:
    - Valores predeterminados:
      - 0 (ilimitado) para las directivas que se establecieron antes del 2020 de junio.
      - 16 millones para las directivas que se establecieron a partir del 2020 de junio.
    - Recuento máximo permitido de filas de la extensión combinada.
    - Se aplica a las operaciones Merge, no a Rebuild.  
- **OriginalSizeMBUpperBoundForMerge**:
    - El valor predeterminado es 0 (ilimitado).
    - Tamaño original máximo permitido (en MB) de la extensión combinada.
    - Se aplica a las operaciones Merge, no a Rebuild.  
- **MaxExtentsToMerge**:
    - El valor predeterminado es 100.
    - Número máximo permitido de extensiones que se van a combinar en una sola operación.
    - Se aplica a las operaciones Merge.
- **LoopPeriod**:
    - El valor predeterminado es 01:00:00 (1 hora).
    - El tiempo máximo de espera entre el inicio de dos iteraciones consecutivas de las operaciones de combinación y recompilación (realizadas por el servicio Administración de datos).
    - Se aplica a las operaciones Merge y Rebuild.
- **AllowRebuild**:
    - El valor predeterminado es "true".
    - Define si `Rebuild` las operaciones están habilitadas (en cuyo caso, son preferibles a `Merge` las operaciones).
- **AllowMerge**:
    - El valor predeterminado es "true".
    - Define si `Merge` las operaciones están habilitadas (en cuyo caso, son menos preferibles que `Rebuild` las operaciones).
- **MaxRangeInHours**:
    - El valor predeterminado es 8.
    - La diferencia máxima permitida (en horas) entre dos horas de creación de dos extensiones diferentes, por lo que todavía se pueden combinar.
    - Las marcas de tiempo son las de la creación de extensiones y no se relacionan con los datos reales contenidos en las extensiones.
    - Se aplica a las operaciones Merge y Rebuild.
    - Un procedimiento recomendado es que este valor se corresponda con el *SoftDeletePeriod*de la [Directiva de retención](./retentionpolicy.md)de la tabla o de la base de datos, o con el *DataHotSpan* de la [Directiva de caché](./cachepolicy.md)(el más bajo de los dos), por lo que se encuentra entre el 2-3% de este último.

**`MaxRangeInHours`Example**

|min (SoftDeletePeriod (Directiva de retención), DataHotSpan (Directiva de caché))|Intervalo máximo en horas (Directiva de combinación)|
|--------------------------------------------------------------------|---------------------------------|
|7 días (168 horas)                                                  | 4                               |
|14 días (336 horas)                                                 | 8                               |
|30 días (720 horas)                                                 | 18                              |
|60 días (1.440 horas)                                               | 36                              |
|90 días (2.160 horas)                                               | 60                              |
|180 días (4.320 horas)                                              | 120                             |
|365 días (8.760 horas)                                              | 250                             |

> [!WARNING]
> Póngase en contacto con el equipo de Azure Explorador de datos antes de modificar una directiva de combinación de extensiones.

Cuando se crea una base de datos, se establece con la Directiva de combinación predeterminada (una directiva con los valores predeterminados mencionados anteriormente), que, de forma predeterminada, es heredada por todas las tablas creadas en la base de datos (a menos que sus directivas se invaliden explícitamente en el nivel de tabla).

Los comandos de control que permiten administrar directivas de fusión mediante combinación para bases de datos o tablas se pueden encontrar [aquí](../management/merge-policy.md).
