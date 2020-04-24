---
title: operador de facetas- Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe el operador de facetas en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0b6c87fc044e0f00f77e28e85d89757a69835164
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515324"
---
# <a name="facet-operator"></a>Operador facet

Devuelve un conjunto de tablas, una para cada columna especificada.
Cada tabla especifica la lista de valores tomados por su columna.
Se puede crear una tabla `with` adicional mediante la cláusula.

**Sintaxis**

*T* `| facet by` *ColumnName* [`, ` ...] [`with (` *filterPipe*`)`

**Argumentos**

* *ColumnName:* El nombre de la columna en la entrada, que se resumirá como una tabla de salida.
* *filterPipe:* Expresión de consulta aplicada a la tabla de entrada para generar una de las salidas.

**Devuelve**

Varias tablas: `with` una para la cláusula y otra para cada columna.

**Ejemplo**

```kusto
MyTable 
| facet by city, eventType 
    with (where timestamp > ago(7d) | take 1000)
```