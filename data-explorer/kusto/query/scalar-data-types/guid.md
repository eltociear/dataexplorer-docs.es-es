---
title: El tipo de datos guid - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe el tipo de datos guid en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/15/2020
ms.openlocfilehash: 382b2da0ab791cf3e2fea0e1c785ee42634aab07
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509867"
---
# <a name="the-guid-data-type"></a>El tipo de datos guid

El `guid` `uuid`tipo `uniqueid`de datos ( , ) representa un valor único global de 128 bits.

> [!WARNING]
> En el momento en que se redactó este documento, la compatibilidad con el tipo `guid` no es completa.
> La brecha principal es la falta de un índice en columnas de este tipo, que afecta al rendimiento de las consultas que predican sobre este tipo.
> Es muy recomendable que los equipos usen valores del tipo `string` en su lugar.

## <a name="guid-literals"></a>guid literales

Para representar un `guid`literal de tipo , utilice el siguiente formato:

```kusto
guid(74be27de-1e4e-49d9-b579-fe0b331d3642)
```

El formulario `guid(null)` especial representa el [valor nulo](null-values.md).