---
title: arg_min() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe arg_min() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/12/2019
ms.openlocfilehash: 4531bd498cf9d9053d57d8b463c7b0adfcc9bf8d
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663986"
---
# <a name="arg_min-aggregation-function"></a>arg_min() (función de agregación)

Busca una fila en el grupo que minimiza *ExprToMinimize*y devuelve el `*` valor de *ExprToReturn* (o para devolver toda la fila).

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

`summarize`[`(`*NameExprToMinimize* `,` *NameExprToReturn* [`,` ...] `)=` `arg_min` ] `(` *ExprToMinimize*, `*`  |  *ExprToReturn* [`,` ...]`)`

**Argumentos**

* *ExprToMinimize*: Expresión que se usará para el cálculo de agregación. 
* *ExprToReturn*: Expresión que se usará para devolver el valor cuando *ExprToMinimize* es mínimo. La expresión que se va a devolver puede ser un carácter comodín (*) para devolver todas las columnas de la tabla de entrada.
* *NameExprToMinimize*: Un nombre opcional para la columna de resultados que representa *ExprToMinimize*.
* *NameExprToReturn*: Nombres opcionales adicionales para las columnas de resultados que representan *ExprToReturn*.

**Devuelve**

Busca una fila en el grupo que minimiza *ExprToMinimize*y devuelve el `*` valor de *ExprToReturn* (o para devolver toda la fila).

**Ejemplos**

Mostrar el proveedor más barato de cada producto:

```kusto
Supplies | summarize arg_min(Price, Supplier) by Product
```

Mostrar todos los detalles, no sólo el nombre del proveedor:

```kusto
Supplies | summarize arg_min(Price, *) by Product
```

Encuentra la ciudad más meridional de cada continente, con su país:

```kusto
PageViewLog 
| summarize (latitude, min_lat_City, min_lat_country)=arg_min(latitude, City, country) 
    by continent
```

:::image type="content" source="images/aggregations/arg-min.png" alt-text="Arg min":::
