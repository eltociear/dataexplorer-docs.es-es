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
ms.openlocfilehash: 7ac5a92b2084eaf2b447f296be34b2f4e79e1bb7
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520135"
---
# <a name="sandbox-policy"></a>Política de espacio aislado

Los siguientes comandos permiten la administración de [entornos limitados](../concepts/sandboxes.md) y [directivas](sandboxpolicy.md) de espacio aislado en un servicio de motor de Kusto.

Los comandos requieren permisos [AllDatabasesAdmin.](access-control/role-based-authorization.md)

## <a name="sandbox-policy"></a>Política de espacio aislado

### <a name="show-cluster-policy-sandbox"></a>Caja de arena de directiva de clúster .show

Muestra todas las directivas de espacio aislado configuradas en el nivel de clúster.

```kusto
.show cluster policy sandbox
```

### <a name="alter-cluster-policy-sandbox"></a>Caja de arena de la directiva de clúster .alter

Modifica la colección de directivas de espacio aislado en el nivel de clúster.

```kusto
.alter cluster policy sandbox @'['
  '{'
    '"SandboxKind": "PythonExecution",'
    '"IsEnabled": true,'
    '"TargetCountPerNode": 4,'
    '"MaxCpuRatePerSandbox": 50,'
    '"MaxMemoryMbPerSandbox": 10240'
  '},'
  '{'
    '"SandboxKind": "RExecution",'
    '"IsEnabled": true,'
    '"TargetCountPerNode": 4,'
    '"MaxCpuRatePerSandbox": 50,'
    '"MaxMemoryMbPerSandbox": 10240'
  '}'
']'
```

### <a name="drop-cluster-policy-sandbox"></a>Caja de arena de directiva de clúster .drop

Para quitar **todas las** directivas de espacio aislado, utilice el siguiente comando:

```kusto
.delete cluster policy sandbox
```

