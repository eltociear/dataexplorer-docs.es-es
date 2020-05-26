---
title: 'Directiva de capacidad: Azure Explorador de datos'
description: En este artículo se describe la Directiva de capacidad de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/12/2020
ms.openlocfilehash: 4a2b18f28840291fc02002239297b45f38be00be
ms.sourcegitcommit: 4e46b497d518884693a142f4ae21ea497db81861
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2020
ms.locfileid: "83824924"
---
# <a name="capacity-policy"></a>Directiva de capacidad

Se usa una directiva de capacidad para controlar los recursos de proceso utilizados para las operaciones de administración de datos en el clúster.

## <a name="the-capacity-policy-object"></a>El objeto de directiva de capacidad

La Directiva de capacidad se compone de:

* [IngestionCapacity](#ingestion-capacity)
* [ExtentsMergeCapacity](#extents-merge-capacity)
* [ExtentsPurgeRebuildCapacity](#extents-purge-rebuild-capacity)
* [ExportCapacity](#export-capacity)
* [ExtentsPartitionCapacity](#extents-partition-capacity)

## <a name="ingestion-capacity"></a>Capacidad de ingesta

|Propiedad                           |Tipo    |Descripción                                                                                                                                                                               |
|-----------------------------------|--------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|ClusterMaximumConcurrentOperations |long    |Un valor máximo para el número de operaciones de ingesta simultáneas en un clúster                                                                                                            |
|CoreUtilizationCoefficient         |double  |Un coeficiente para el porcentaje de núcleos que se va a usar al calcular la capacidad de ingesta (el resultado del cálculo siempre será normalizado por `ClusterMaximumConcurrentOperations` ). |                                                                                                                             |

La capacidad total de ingesta del clúster (como se muestra en [. show Capacity](../management/diagnostics.md#show-capacity)) se calcula mediante:

Mínimo ( `ClusterMaximumConcurrentOperations` , `Number of nodes in cluster` * máximo (1, `Core count per node`  *  `CoreUtilizationCoefficient` ))

> [!Note]
> En clústeres con tres o más nodos, el nodo de administración no participa en realizar operaciones de ingesta. `Number of nodes in cluster`Se reduce en uno.

## <a name="extents-merge-capacity"></a>Capacidad de combinación de extensiones

|Propiedad                           |Tipo    |Descripción                                                                                    |
|-----------------------------------|--------|-----------------------------------------------------------------------------------------------|
|MaximumConcurrentOperationsPerNode |long    |Un valor máximo para el número de operaciones de combinación o recompilación de extensiones simultáneas en un único nodo |

La capacidad de combinación de extensiones totales del clúster (como se muestra en [. Mostrar capacidad](../management/diagnostics.md#show-capacity)) se calcula mediante:

`Number of nodes in cluster`x1`MaximumConcurrentOperationsPerNode`

> [!Note]
> * `MaximumConcurrentOperationsPerNode`la ajusta automáticamente el sistema en el intervalo [1, 5], a menos que se haya establecido en un valor mayor.
> * En clústeres con tres o más nodos, el nodo de administración no participa en las operaciones de combinación. `Number of nodes in cluster`Se reduce en uno.

## <a name="extents-purge-rebuild-capacity"></a>Extensiones purgar capacidad de reconstrucción

|Propiedad                           |Tipo    |Descripción                                                                                                                           |
|-----------------------------------|--------|--------------------------------------------------------------------------------------------------------------------------------------|
|MaximumConcurrentOperationsPerNode |long    |Un valor máximo para el número de extensiones de recompilación simultáneas para las operaciones de purga en un nodo único. |

Las extensiones totales del clúster purgan la capacidad de recompilación (como se muestra en [. Mostrar capacidad](../management/diagnostics.md#show-capacity)) se calculan mediante:

`Number of nodes in cluster`x1`MaximumConcurrentOperationsPerNode`

> [!Note]
> En clústeres con tres o más nodos, el nodo de administración no participa en las operaciones de combinación. `Number of nodes in cluster`Se reduce en uno.

## <a name="export-capacity"></a>Capacidad de exportación

|Propiedad                           |Tipo    |Descripción                                                                                                                                                                            |
|-----------------------------------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|ClusterMaximumConcurrentOperations |long    |Un valor máximo para el número de operaciones de exportación simultáneas en un clúster.                                                                                                           |
|CoreUtilizationCoefficient         |double  |Un coeficiente para el porcentaje de núcleos que se va a usar al calcular la capacidad de exportación. El resultado del cálculo siempre será normalizado por `ClusterMaximumConcurrentOperations` . |

La capacidad total de exportación del clúster (como se muestra en [. show Capacity](../management/diagnostics.md#show-capacity)) se calcula mediante:

Mínimo ( `ClusterMaximumConcurrentOperations` , `Number of nodes in cluster` * máximo (1, `Core count per node`  *  `CoreUtilizationCoefficient` ))

> [!Note]
> En clústeres con tres o más nodos, el nodo de administración no participa en las operaciones de exportación. `Number of nodes in cluster`Se reduce en uno.

## <a name="extents-partition-capacity"></a>Capacidad de partición de extensiones

|Propiedad                           |Tipo    |Descripción                                                                             |
|-----------------------------------|--------|----------------------------------------------------------------------------------------|
|ClusterMaximumConcurrentOperations |long    |Un valor máximo para el número de operaciones de partición de extensiones simultáneas en un clúster. |

La capacidad de partición de extensiones totales del clúster (como se muestra en [. Mostrar capacidad](../management/diagnostics.md#show-capacity)) se define mediante una sola propiedad: `ClusterMaximumConcurrentOperations` .

> [!Note]
> `ClusterMaximumConcurrentOperations`la ajusta automáticamente el sistema en el intervalo [1, 16], a menos que se haya establecido en un valor mayor.

## <a name="defaults"></a>Valores predeterminados

La Directiva de capacidad predeterminada tiene la siguiente representación JSON:

```kusto 
{
  "IngestionCapacity": {
    "ClusterMaximumConcurrentOperations": 512,
    "CoreUtilizationCoefficient": 0.75
  },
  "ExtentsMergeCapacity": {
    "MaximumConcurrentOperationsPerNode": 1
  },
  "ExtentsPurgeRebuildCapacity": {
    "MaximumConcurrentOperationsPerNode": 1
  },
  "ExportCapacity": {
    "ClusterMaximumConcurrentOperations": 100,
    "CoreUtilizationCoefficient": 0.25
  }
}
```

## <a name="control-commands"></a>Comandos de control

> [!WARNING]
> Póngase en contacto con el equipo de Azure Explorador de datos antes de modificar una directiva de capacidad.

* Use [. Mostrar la capacidad](capacity-policy.md#show-cluster-policy-capacity) de la Directiva de clúster para mostrar la Directiva de capacidad actual del clúster.

* Use [. Alter la capacidad](capacity-policy.md#alter-cluster-policy-capacity) de la Directiva de clúster para modificar la Directiva de capacidad del clúster.

## <a name="throttling"></a>Limitaciones

Kusto limita el número de solicitudes simultáneas para los siguientes comandos iniciados por el usuario:

* Ingesta (incluye todos los comandos que se enumeran [aquí](../management/data-ingestion/index.md))
   * El límite es como se define en la [Directiva de capacidad](#capacity-policy).
* Purga
   * Global está actualmente fijo en uno por clúster.
   * La capacidad de recompilación de purga se usa internamente para determinar el número de operaciones de recompilación simultáneas durante los comandos de purga. Los comandos de purga no se bloquearán ni limitarán debido a este proceso, pero funcionarán más rápido o más lentamente en función de la capacidad de recompilación de purga.
* Exports
   * El límite es como se define en la [Directiva de capacidad](#capacity-policy).

Cuando el clúster detecta que una operación ha superado la operación simultánea permitida, responderá con un código HTTP 429 ("limitado").
Vuelva a intentar la operación después de un retroceso.
