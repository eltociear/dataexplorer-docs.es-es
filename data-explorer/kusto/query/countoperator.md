---
title: 'operador de recuento: Explorador de datos de Azure . . . . . . . . . . . . . . Microsoft Docs'
description: En este artículo se describe el operador count en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/16/2020
ms.openlocfilehash: c0d0286919a68b6e58065e0a6fe7e0d24b1cdd5f
ms.sourcegitcommit: c4aea69fafa9d9fbb814764eebbb0ae93fa87897
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2020
ms.locfileid: "81610226"
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

```kusto
StormEvents | count
```
