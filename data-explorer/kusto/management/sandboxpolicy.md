---
title: 'Directiva de espacio aislado: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la directiva de espacio aislado en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/24/2020
ms.openlocfilehash: 0a59e25c6c38d3189330299af1b19f89357cb456
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520067"
---
# <a name="sandbox-policy"></a>Política de espacio aislado

## <a name="overview"></a>Información general

Kusto admite la ejecución de ciertos plugins dentro de [entornos limitados,](../concepts/sandboxes.md)donde los recursos disponibles para el sandbox son limitados y controlados, tanto por motivos de seguridad, como para fines de gobierno de recursos.

Los espacios aislados se ejecutan en los nodos del servicio de motor de Kusto y algunas de sus limitaciones se definen en las directivas de espacio aislado, donde cada tipo de espacio aislado puede tener su propia directiva de espacio aislado.

Las directivas de espacio aislado se administran a nivel de clúster y afectan a todos los nodos del clúster.

La modificación de las directivas requiere permisos [AllDatabasesAdmin.](../management/access-control/role-based-authorization.md)

## <a name="the-policy-object"></a>El objeto de política

una directiva de espacio aislado tiene las siguientes propiedades:

* **SandboxKind**: define el tipo de sandbox `PythonExecution` `RExecution`(por ejemplo, , ).
* **IsEnabled**: define si los entornos limitados de este tipo están habilitados para ejecutarse en los nodos del clúster.
* **TargetCountPerNode**: define cuántos entornos limitados de este tipo se pueden ejecutar en los nodos del clúster.
  * Los valores pueden estar entre 1 y el doble del número de procesadores por nodo.
  * El valor predeterminado es `16`.
* **MaxCpuRatePerSandbox**: define la velocidad máxima de CPU en porcentaje de todos los núcleos disponibles que un solo sandbox puede utilizar.
  * Los valores pueden estar entre 1 y 100.
  * El valor predeterminado es `50`.
* **MaxMemoryMbPerSandbox**: define la cantidad máxima de memoria (en megabytes) que puede usar un único entorno limitado.
  * Los valores pueden estar entre 200 y 65536 (64 GB).
  * El valor `20480` predeterminado es (20 GB).

## <a name="example"></a>Ejemplo

La siguiente política establece límites diferentes para `PythonExecution` 2 `RExecution`tipos diferentes de entornos limitados, y:

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

## <a name="notes"></a>Notas

* Los cambios en la directiva de espacio aislado se aplican a las cajas de sanbox creadas a partir del momento en que se aplica el cambio.
  * Los espacios aislados que se han asignado previamente antes del cambio de directiva seguirán ejecutándose de acuerdo con los límites de directiva anteriores, hasta que se utilicen como parte de una consulta.
* Podría haber un retraso de hasta 5 minutos hasta que el cambio en la directiva surta efecto, ya que los nodos del clúster sondean periódicamente los cambios de directiva.

## <a name="next-steps"></a>Pasos siguientes

Utilice los comandos de control de [políticas](../management/sandbox-policy.md) de espacio aislado para administrar la directiva de espacio aislado del clúster.