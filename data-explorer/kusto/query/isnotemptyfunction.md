---
title: 'isnotempty (): Azure Explorador de datos'
description: En este artículo se describe isnotempty () en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 5a21031b07df3a4fa654fd13eb3761618308337b
ms.sourcegitcommit: ae72164adc1dc8d91ef326e757376a96ee1b588d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2020
ms.locfileid: "84717349"
---
# <a name="isnotempty"></a>isnotempty()

Devuelve `true` si el argumento no es una cadena vacía y no es NULL.

```kusto
isnotempty("") == false
```

**Sintaxis**

`isnotempty(`[*valor*]`)`

`notempty(`[*valor*] `)` --alias de`isnotempty`
