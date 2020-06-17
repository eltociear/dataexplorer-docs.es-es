---
title: 'Directiva de combinación de extensiones: Azure Explorador de datos'
description: En este artículo se describen las extensiones de la Directiva de combinación de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 1f0a95e095dfaa11afdea3b66597ff3d98f0449b
ms.sourcegitcommit: 3848b8db4c3a16bda91c4a5b7b8b2e1088458a3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/16/2020
ms.locfileid: "84818620"
---
# <a name="extents-merge-policy"></a>Directiva de combinación de extensiones

La Directiva de combinación define si se deben combinar las [extensiones (particiones de datos)](../management/extents-overview.md) en el clúster de Kusto.

Hay dos tipos de operaciones de combinación: `Merge` , que vuelve a generar los índices, y `Rebuild` , que reingerin por completo los datos.

Ambos tipos de operación dan como resultado una única extensión que reemplaza las extensiones de origen.

De forma predeterminada, se prefieren las operaciones de recompilación. Si hay extensiones que no se ajustan a los criterios para que se vuelvan a generar, se realizará un intento para combinarlas.  

> [!NOTE]
> * El etiquetado de extensiones mediante etiquetas *diferentes* hará que `drop-by` no se combinen estas extensiones, incluso si se ha establecido una directiva de combinación. Para obtener más información, consulte [etiquetado de extensiones](../management/extents-overview.md#extent-tagging).
> * No se combinarán las extensiones cuya unión de etiquetas supere la longitud de 1 millón de caracteres.
> * La [Directiva de particionamiento](./shardingpolicy.md) de la base de datos o de la tabla también tiene algún efecto sobre cómo se mezclan las extensiones.

## <a name="merge-policy-properties"></a>Propiedades de la Directiva de combinación

La Directiva de combinación contiene las siguientes propiedades:

* **RowCountUpperBoundForMerge**:
    * Valores predeterminados:
      * 0 (ilimitado) para las directivas que se establecieron antes del 2020 de junio.
      * 16 millones para las directivas que se establecieron a partir del 2020 de junio.
    * Recuento máximo permitido de filas de la extensión combinada.
    * Se aplica a las operaciones Merge, no a Rebuild.  
* **OriginalSizeMBUpperBoundForMerge**:
    * El valor predeterminado es 0 (ilimitado).
    * Tamaño original máximo permitido (en MB) de la extensión combinada.
    * Se aplica a las operaciones Merge, no a Rebuild.  
* **MaxExtentsToMerge**:
    * El valor predeterminado es 100.
    * Número máximo permitido de extensiones que se van a combinar en una sola operación.
    * Se aplica a las operaciones Merge.
* **LoopPeriod**:
    * El valor predeterminado es 01:00:00 (1 hora).
    * El tiempo máximo de espera entre el inicio de dos iteraciones consecutivas de las operaciones de combinación o recompilación por parte del servicio de Administración de datos.
    * Se aplica a las operaciones Merge y Rebuild.
* **AllowRebuild**:
    * El valor predeterminado es "true".
    * Define si `Rebuild` las operaciones están habilitadas (en cuyo caso, son preferibles a `Merge` las operaciones).
* **AllowMerge**:
    * El valor predeterminado es "true".
    * Define si `Merge` las operaciones están habilitadas, en cuyo caso son menos preferibles que `Rebuild` las operaciones.
* **MaxRangeInHours**:
    * El valor predeterminado es 8.
    * Diferencia máxima permitida, en horas, entre dos horas de creación de dos extensiones diferentes, de modo que todavía se pueden combinar.
    * Las marcas de tiempo son de la creación de extensiones y no se relacionan con los datos reales contenidos en las extensiones.
    * Se aplica a las operaciones Merge y Rebuild.
    * Este valor debe establecerse según la Directiva de [retención](./retentionpolicy.md) *vigente SoftDeletePeriod*o los valores de *DataHotSpan* de la [Directiva de caché](./cachepolicy.md) . Tome el valor más bajo de *SoftDeletePeriod* y *DataHotSpan*. Establezca el valor de *MaxRangeInHours* en entre el 2-3% del mismo. Vea los [ejemplos](#maxrangeinhours-examples) .

## <a name="maxrangeinhours-examples"></a>Ejemplos de MaxRangeInHours

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

Cuando se crea una base de datos, se establece con los valores de directiva de combinación predeterminados mencionados anteriormente. De forma predeterminada, la Directiva la heredan todas las tablas creadas en la base de datos, a menos que sus directivas se invaliden explícitamente en el nivel de tabla.

Para obtener más información, vea [comandos de control que permiten administrar directivas de combinación para bases de datos o tablas](../management/merge-policy.md).
