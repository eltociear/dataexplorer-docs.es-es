---
title: repeat() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe repeat() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0fd944112a64e7400e38c627b7b0651bb6fccd54
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510428"
---
# <a name="repeat"></a>repeat()

Genera una matriz dinámica que contiene una serie de valores iguales.

**Sintaxis**

`repeat(`*value*`,` *conteo de* valor`)` 

**Argumentos**

* *valor*: el valor del elemento de la matriz resultante. El tipo de *valor* puede ser booleano, entero, largo, real, datetime o intervalo de tiempo.   
* *count*: El recuento de los elementos de la matriz resultante. El *recuento* debe ser un número entero.
Si *count* es igual a cero, se devuelve una matriz vacía.
Si *count* es menor que cero, se devuelve un valor nulo. 

**Ejemplos**

El siguiente ejemplo devuelve `[1, 1, 1]`:

```kusto
T | extend r = repeat(1, 3)
```