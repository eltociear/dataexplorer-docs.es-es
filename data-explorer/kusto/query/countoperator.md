---
title: 'operador Count: Azure Explorador de datos'
description: En este artículo se describe el operador de recuento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/16/2020
ms.openlocfilehash: 9a34734ebfee94646b2b2f15730f14f9d2709c6d
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83227523"
---
# <a name="count-operator"></a>Operador count

Devuelve el número de registros del conjunto de registros de entrada.

**Sintaxis**

`T | count`

**Argumentos**

*T*: los datos tabulares cuyos registros se van a contabilizar.

**Devuelve**

Esta función devuelve una tabla con un único registro y una columna de tipo `long`. El valor de la celda única es el número de registros en *T*. 

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
StormEvents | count
```
