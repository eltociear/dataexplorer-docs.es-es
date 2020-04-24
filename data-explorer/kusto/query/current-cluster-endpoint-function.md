---
title: current_cluster_endpoint() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe current_cluster_endpoint() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 65ce130b4dd3e0a3125eefc6c410775647f9b964
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516854"
---
# <a name="current_cluster_endpoint"></a>current_cluster_endpoint()

Devuelve el punto de conexión de red (nombre DNS) del clúster actual que se está consultando.

**Sintaxis**

`current_cluster_endpoint()`

**Devuelve**

El punto de conexión de red (nombre DNS) del `string`clúster actual que se está consultando, como un valor de tipo .

**Ejemplo**

```kusto
print strcat("This query executed on: ", current_cluster_endpoint())
```