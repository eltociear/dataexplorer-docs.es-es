---
title: 'Comandos de control de directivas de capacidad: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describen los comandos de control de directivas de capacidad en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/02/2020
ms.openlocfilehash: 929cfa885a7373b400b832d908677a7a5fb93ef6
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522090"
---
# <a name="capacity-policy-control-commands"></a>Comandos de control de políticas de capacidad

Los siguientes comandos de control se pueden utilizar para administrar la directiva de [capacidad](../management/capacitypolicy.md)de un clúster.

Los comandos requieren permisos [AllDatabasesAdmin.](../management/access-control/role-based-authorization.md)

## <a name="show-cluster-policy-capacity"></a>mostrar la capacidad de la política de clúster

```kusto
.show cluster policy capacity
```

Muestra la directiva de capacidad actual para el clúster.

**Salida**

|Nombre de la directiva | Nombre de entidad | Directiva | Entidades secundarias | Tipo de entidad
|---|---|---|---|---
|CapacityPolicy | | Cadena con formato JSON que representa la política | La lista de bases de datos en el clúster |Clúster


## <a name="alter-cluster-policy-capacity"></a>alterar la capacidad de la política de clúster

```kusto
.alter cluster policy capacity @'{ ... capacity policy JSON representation ... }'
.alter-merge cluster policy capacity @'{ ... capacity policy partial-JSON representation ... }'
```

**Nota:** Los cambios en la directiva de capacidad del clúster podrían tardar hasta 1 hora en surtir efecto.

**Ejemplos:**

* Modificación explícita de todas las propiedades de la directiva de clúster:

```kusto
.alter cluster policy capacity
'{'
  '"IngestionCapacity": {'
    '"ClusterMaximumConcurrentOperations": 512,'
    '"CoreUtilizationCoefficient": 0.75'
  '},'
  '"ExtentsMergeCapacity": {'
    '"MaximumConcurrentOperationsPerNode": 1'
  '},'
  '"ExtentsPurgeRebuildCapacity": {'
    '"MaximumConcurrentOperationsPerNode": 1'
  '},'
  '"ExportCapacity": {'
    '"ClusterMaximumConcurrentOperations": 100,'
    '"CoreUtilizationCoefficient": 0.25'
  '},'
  '"ExtentsPartitionCapacity": {'
    '"ClusterMaximumConcurrentOperations": 4'
  '}'
'}'
```

* Modificar una sola propiedad de la directiva de nivel de clúster, mantener todas las demás propiedades intactas:

```kusto
.alter-merge cluster policy capacity
'{'
  '"ExtentsPartitionCapacity": {'
    '"MaximumConcurrentOperationsPerNode": 4'
  '}'
'}'
```
