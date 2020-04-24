---
title: complemento de vista previa - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe el complemento de vista previa en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 18bda0a4348d0c0eb2776bf124c57397f318a989
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510989"
---
# <a name="preview-plugin"></a>plugin de vista previa

Devuelve una tabla con hasta el número especificado de filas del conjunto de registros de entrada y el número total de registros del conjunto de registros de entrada.

```kusto
T | evaluate preview(50)
```

**Sintaxis**

`T` `|` `evaluate` `preview(` *NumberOfRows* `)`

**Devuelve**

El `preview` plugin devuelve dos tablas de resultados:
* Una tabla con hasta el número especificado de filas.
  Por ejemplo, la consulta de `T | take 50`ejemplo anterior es equivalente a ejecutar .
* Una tabla con una sola fila/columna, que contiene el número de registros en el conjunto de registros de entrada.
  Por ejemplo, la consulta de `T | count`ejemplo anterior es equivalente a ejecutar .

**Sugerencias**

Si `evaluate` está precedido por un origen tabular que incluye un filtro complejo, o un [`materialize`](materializefunction.md) filtro que hace referencia a la mayoría de las columnas de la tabla de origen, prefiere utilizar la función. Por ejemplo:

```kusto
let MaterializedT = materialize(T);
MaterializedT | evaluate preview(50)
```