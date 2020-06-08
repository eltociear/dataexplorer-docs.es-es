---
title: 'Directiva de espacio aislado: Azure Explorador de datos'
description: En este artículo se describe la Directiva de espacio aislado en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 786f771878a7216b62dce127391f0e7967e954f6
ms.sourcegitcommit: 188f89553b9d0230a8e7152fa1fce56c09ebb6d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2020
ms.locfileid: "84512460"
---
# <a name="sandbox-policy"></a>Directiva de espacio aislado

Azure Explorador de datos ejecuta determinados complementos en [espacios aislados](../concepts/sandboxes.md) cuyos recursos disponibles se limitan y controlan para la seguridad y la regulación de los recursos.

Los espacios aislados se ejecutan en los nodos del motor Kusto. Algunas de sus limitaciones se definen en las directivas de espacio aislado, donde cada tipo de espacio aislado puede tener su propia Directiva.

Las directivas de espacio aislado se administran en el nivel de clúster y afectan a todos los nodos del clúster.

Para modificar las directivas, necesitará permisos de [AllDatabasesAdmin](../management/access-control/role-based-authorization.md) .

## <a name="the-policy-object"></a>El objeto de Directiva

Una directiva de espacio aislado tiene las siguientes propiedades.

* **SandboxKind**: define el tipo de espacio aislado (por ejemplo, `PythonExecution` , `RExecution` ).
* **IsEnabled**: define si los espacios aislados de este tipo se pueden ejecutar en los nodos del clúster.
* **TargetCountPerNode**: define cuántos espacios aislados de este tipo pueden ejecutarse en los nodos del clúster.
  * Los valores pueden estar entre uno y dos veces el número de procesadores por nodo.
  * El valor predeterminado es 16.
* **MaxCpuRatePerSandbox**: define la velocidad máxima de la CPU como un porcentaje de todos los núcleos disponibles que puede usar un solo espacio aislado.
  * Los valores pueden estar entre 1 y 100.
  * El valor predeterminado es 50.
* **MaxMemoryMbPerSandbox**: define la cantidad máxima de memoria (en megabytes) que puede usar un solo espacio aislado.
  * Los valores pueden estar entre 200 y 65536 (64 GB).
  * El valor predeterminado es 20480 (20 GB).

## <a name="example"></a>Ejemplo

La siguiente Directiva establece límites diferentes para `PythonExecution` los `RExecution` espacios aislados y:

```json
[
  {
    "SandboxKind": "PythonExecution",
    "IsEnabled": true,
    "TargetCountPerNode": 4,
    "MaxCpuRatePerSandbox": 55,
    "MaxMemoryMbPerSandbox": 65536
  },
  {
    "SandboxKind": "RExecution",
    "IsEnabled": true,
    "TargetCountPerNode": 2,
    "MaxCpuRatePerSandbox": 50,
    "MaxMemoryMbPerSandbox": 10240
  }
]
```

> [!NOTE]
> * Los cambios en la Directiva de espacio aislado se aplican a los espacios aislados creados a partir de la hora a la que se aplica el cambio. Los espacios aislados que se han asignado previamente antes del cambio de la Directiva seguirán ejecutándose según los límites de la directiva anterior, hasta que se usen como parte de una consulta.
> * Podría haber un retraso de hasta cinco minutos hasta que el cambio de la Directiva surta efecto, ya que los nodos del clúster sondean periódicamente los cambios de la Directiva.

## <a name="next-steps"></a>Pasos siguientes

Use los [comandos de control de directiva de espacio aislado](../management/sandbox-policy.md) para administrar la Directiva de espacio aislado del clúster.
 
