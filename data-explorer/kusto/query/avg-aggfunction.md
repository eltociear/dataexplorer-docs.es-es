---
title: avg() (función de agregación) - Explorador de datos de Azure ? Microsoft Docs
description: En este artículo se describe avg() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/26/2019
ms.openlocfilehash: aadc756bdf4c6cab805f58a8a600815cf29680f7
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518350"
---
# <a name="avg-aggregation-function"></a>avg() (función de agregación)

Calcula el promedio de *Expr* en todo el grupo. 

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

**Sintaxis**

resumir `avg(` *Expr*`)`

**Argumentos**

* *Expr*: Expresión que se utilizará para el cálculo de la agregación. Los `null` registros con valores se omiten y no se incluyen en el cálculo.

**Devuelve**

El valor medio de *Expr* en todo el grupo.
 