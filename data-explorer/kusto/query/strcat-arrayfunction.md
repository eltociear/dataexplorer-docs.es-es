---
title: strcat_array() - Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe strcat_array() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 2d2412762cf68243e3952a8ad12a5b919d947bd3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506960"
---
# <a name="strcat_array"></a>strcat_array()

Crea una cadena concatenada de valores de matriz utilizando el delimitador especificado.
    
**Sintaxis**

`strcat_array(`*array*, *delimitador*`)`

**Argumentos**

* *array*: `dynamic` un valor que representa una matriz de valores que se van a concatenar.
* *delímetro*: `string` Un valor que se utilizará para concatenar los valores en *la matriz*

**Devuelve**

Valores de matriz, concatenados en una sola cadena.

**Ejemplos**
  
```kusto
print str = strcat_array(dynamic([1, 2, 3]), "->")
```

|str|
|---|
|1->2->3|