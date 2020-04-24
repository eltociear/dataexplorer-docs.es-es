---
title: 'Directiva de capacidad: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la directiva de capacidad en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/12/2020
ms.openlocfilehash: af648bd0a4b328477b14e20a2457e3e914df2827
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522005"
---
# <a name="capacity-policy"></a>Política de capacidad

Una directiva de capacidad se utiliza para controlar los recursos informáticos que se utilizan para realizar la ingesta de datos y otras operaciones de aseo de datos (como la combinación de extensiones).

## <a name="the-capacity-policy-object"></a>El objeto de política de capacidad

La política de capacidad `IngestionCapacity` `ExtentsMergeCapacity`se `ExtentsPurgeRebuildCapacity` `ExportCapacity`compone de , , y .

### <a name="ingestion-capacity"></a>Capacidad de ingestión

|Propiedad                           |Tipo    |Descripción                                                                                                                                                                               |
|-----------------------------------|--------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|ClusterMaximumConcurrentOperations |long    |Un valor máximo para el número de operaciones de ingesta simultáneas en un clúster                                                                                                            |
|CoreUtilizationCoefficient         |double  |Un coeficiente para el porcentaje de núcleos que se utilizarán al calcular la capacidad `ClusterMaximumConcurrentOperations`de ingesta (el resultado del cálculo siempre se normalizará mediante ) |                                                                                                                             |

La capacidad total de ingesta del clúster (como se muestra en [la capacidad .show)](../management/diagnostics.md#show-capacity)se calcula mediante:

Mínimo(`ClusterMaximumConcurrentOperations` `Number of nodes in cluster` , * Máximo(1, `Core count per node`  *  `CoreUtilizationCoefficient`))

> [!Note] 
> En clústeres con tres nodos o superior, el nodo admin no `Number of nodes in cluster` participa en la realización de operaciones de ingesta, por lo que se reduce en 1.

### <a name="extents-merge-capacity"></a>Capacidad de fusión de extensiones

|Propiedad                           |Tipo    |Descripción                                                                                    |
|-----------------------------------|--------|-----------------------------------------------------------------------------------------------|
|MaximumConcurrentOperationsPerNode |long    |Un valor máximo para el número de operaciones de fusión/reconstrucción de extensiones simultáneas en un solo nodo |

La capacidad de fusión de extensiones totales del clúster (como se muestra en [la capacidad .show)](../management/diagnostics.md#show-capacity)se calcula mediante:

`Number of nodes in cluster`X`MaximumConcurrentOperationsPerNode`

> [!Note] 
> En clústeres con tres nodos o superior, el nodo admin no `Number of nodes in cluster` participa en la realización de operaciones de combinación, por lo que se reduce en 1.

### <a name="extents-purge-rebuild-capacity"></a>Capacidad de reconstrucción de la purga de extensión

|Propiedad                           |Tipo    |Descripción                                                                                                                           |
|-----------------------------------|--------|--------------------------------------------------------------------------------------------------------------------------------------|
|MaximumConcurrentOperationsPerNode |long    |Un valor máximo para el número de operaciones de reconstrucción de purga de extensiones simultáneas (reconstruir extensiones para operaciones de purga) en un único nodo |

La capacidad de reconstrucción de purga de extensiones totales del clúster (como se muestra en [la capacidad .show)](../management/diagnostics.md#show-capacity)se calcula mediante:

`Number of nodes in cluster`X`MaximumConcurrentOperationsPerNode`

> [!Note] 
> En clústeres con tres nodos o superior, el nodo admin no `Number of nodes in cluster` participa en la realización de operaciones de combinación, por lo que se reduce en 1.

### <a name="export-capacity"></a>Capacidad de exportación

|Propiedad                           |Tipo    |Descripción                                                                                                                                                                            |
|-----------------------------------|--------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|ClusterMaximumConcurrentOperations |long    |Valor máximo para el número de operaciones de exportación simultáneas en un clúster.                                                                                                           |
|CoreUtilizationCoefficient         |double  |Un coeficiente para el porcentaje de núcleos que se utilizarán al calcular la `ClusterMaximumConcurrentOperations`capacidad de exportación (el resultado del cálculo siempre se normalizará mediante ) |

La capacidad total de exportación del clúster (como se muestra en [la capacidad .show)](../management/diagnostics.md#show-capacity)se calcula mediante:

Mínimo(`ClusterMaximumConcurrentOperations` `Number of nodes in cluster` , * Máximo(1, `Core count per node`  *  `CoreUtilizationCoefficient`))

> [!Note] 
> En clústeres con tres nodos o superior, el nodo admin no `Number of nodes in cluster` participa en la realización de operaciones de exportación, por lo que se reduce en 1.

### <a name="extents-partition-capacity"></a>Capacidad de partición de extensiones

|Propiedad                           |Tipo    |Descripción                                                                             |
|-----------------------------------|--------|----------------------------------------------------------------------------------------|
|ClusterMaximumConcurrentOperations |long    |Valor máximo para el número de operaciones de partición de extensiones simultáneas en un clúster. |

La capacidad total de partición de extensiones del clúster (como se `ClusterMaximumConcurrentOperations`muestra en la capacidad [.show)](../management/diagnostics.md#show-capacity)se define mediante una sola propiedad: .

### <a name="defaults"></a>Valores predeterminados

La política de capacidad predeterminada tiene la siguiente representación JSON:

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

> [!WARNING]
> Rara **vez** se recomienda modificar una política de capacidad sin consultar primero con el equipo de Kusto.

## <a name="control-commands"></a>Comandos de control

* Utilice la capacidad de [directiva de clúster .show](capacity-policy.md#show-cluster-policy-capacity) para mostrar la directiva de capacidad actual del clúster.
* Utilice la capacidad de directiva de [clúster .alter](capacity-policy.md#alter-cluster-policy-capacity) para modificar la directiva de capacidad del clúster.

## <a name="throttling"></a>Limitaciones

Kusto limita el número de solicitudes simultáneas para los siguientes comandos:

1. Ingestiones (incluye todos los comandos que se enumeran [aquí)](../management/data-ingestion/index.md)
      * El límite es el que se define en la directiva de [capacidad.](#capacity-policy)
1. Combinaciones
      * El límite es el que se define en la directiva de [capacidad.](#capacity-policy)
1. Purga
      * Global se fija actualmente en uno por clúster.
      * La capacidad de reconstrucción de purga se utiliza internamente para determinar el número de operaciones de reconstrucción simultáneas durante los comandos de purga (los comandos de purga no se bloquearán ni limitarán debido a esto, pero funcionarán más rápido o más lento en función de la capacidad de reconstrucción de purga).
1. Exports
      * El límite es el que se define en la directiva de [capacidad.](#capacity-policy)


Cuando Kusto detecta que alguna operación ha superado la operación simultánea permitida, Kusto responderá con un código HTTP 429.
El cliente debe volver a intentar la operación después de algún retroceso.