---
title: max() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe max() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: bbfc9591fb20903d18486f9f249d3b1240f705e3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512570"
---
# <a name="max-aggregation-function"></a>max() (función de agregación)

Devuelve el valor máximo en todo el grupo. 

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

`summarize``max(` *Expr*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. 

**Devuelve**

El valor máximo de *Expr* en todo el grupo.
 
> [!TIP]
> Esto le da el mínimo o máximo por sí mismo - por ejemplo, el precio más alto o más bajo.
> Pero si desea otras columnas en la fila - por ejemplo, el nombre del proveedor con el precio más bajo - utilizar [arg_max](arg-max-aggfunction.md) o [arg_min](arg-min-aggfunction.md).