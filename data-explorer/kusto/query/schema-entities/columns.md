---
title: Columnas - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describen las columnas del Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 17406eb94f8e29f4b8ab87653ab41df737fa15ef
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509493"
---
# <a name="columns"></a>Columnas

Cada [tabla](tables.md) de Kusto, y cada secuencia de datos tabulares, es una cuadrícula rectangular de columnas y filas. Cada columna de la tabla tiene un nombre y un tipo de [datos escalares](../scalar-data-types/index.md)específico. Las columnas de una tabla o una secuencia de datos tabulares están ordenadas, por lo que una columna también tiene una posición específica en la colección de columnas de la tabla.

**Notas**  

* Los nombres de columna distinguen mayúsculas de minúsculas.
* Los nombres de columna siguen las reglas para los nombres de [entidad.](./entity-names.md)
* El límite máximo de columnas por tabla es de 10.000.

En las consultas, las columnas suelen ser referencias solo por nombre. Solo pueden aparecer en expresiones y el operador de consulta en el que aparece la expresión determina la tabla o la secuencia de datos tabulares, por lo que no es necesario que el nombre de la columna tenga un ámbito adicional. Por ejemplo, en la siguiente consulta tenemos una secuencia de datos tabulars sin nombre (definida a través del operador de tabla de [datos](../datatableoperator.md) que tiene una sola columna, `c`. A continuación, un predicado filtra la secuencia de datos tabulares en el valor de esa columna, lo que genera una nueva secuencia de datos tabulares sin nombre con las mismas columnas pero menos filas. A continuación, el [operador as](../asoperator.md) nombra la secuencia de datos tabulares y su valor se devuelve como resultados de la consulta.
Observe, en particular, cómo se hace referencia a la columna `c` por su nombre sin necesidad de hacer referencia a su contenedor (de hecho, ese contenedor no tiene nombre):

```kusto
datatable (c:int) [int(-1), 0, 1, 2, 3]
| where c*c >= 2
| as Result
```

> Como suele ser común en el mundo de las bases de datos relacionales, las filas a veces se denominan **registros** y las columnas a veces se denominan **atributos.**

Los detalles sobre la administración de columnas se pueden encontrar en [La administración de columnas.](../../management/columns.md)