---
title: make_timespan() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe make_timespan() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 31301f684ea700cf89e9234c4c43adab068319b7
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512774"
---
# <a name="make_timespan"></a>make_timespan()

Crea un valor escalar de intervalo de [tiempo](./scalar-data-types/timespan.md) a partir del período de tiempo especificado.

```kusto
make_timespan(1,12,30,55.123) == time(1.12:30:55.123)
```

**Sintaxis**

`make_timespan(`*hora,**minuto*`)`

`make_timespan(`*hora,**minuto,**segundo*`)`

`make_timespan(`*día,**hora,**minuto,**segundo*`)`

**Argumentos**

* *día*: día (un valor entero positivo)
* *hora*: hora (un valor entero, de 0 a 23)
* *minuto*: minuto (un valor entero, de 0 a 59)
* *segundo:* segundo (un valor real, de 0 a 59.9999999)

**Devuelve**

Si la creación se realiza correctamente, el resultado será un valor de intervalo de [tiempo,](./scalar-data-types/timespan.md) de lo contrario, el resultado será null.
 
**Ejemplo**

```kusto
print ['timespan'] = make_timespan(1,12,30,55.123)

```

|timespan|
|---|
|1.12:30:55.1230000|


