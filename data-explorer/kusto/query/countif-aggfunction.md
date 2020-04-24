---
title: countif() (función de agregación) - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe countif() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 03b6f1cb959706463a73d8aa18a11144e2123492
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516990"
---
# <a name="countif-aggregation-function"></a>countif() (función de agregación)

Devuelve el número de filas en las que *Predicate* se evalúa en `true`.

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)

Vea también - [función count(),](count-aggfunction.md) que cuenta filas sin expresión de predicado.

**Sintaxis**

resumir `countif(` *Predicado*`)`

**Argumentos**

* *Predicado*: Expresión que se utilizará para el cálculo de la agregación. 

**Devuelve**

Devuelve el número de filas en las que *Predicate* se evalúa en `true`.

> [!TIP]
> Usar `summarize countif(filter)` en lugar de `where filter | summarize count()`