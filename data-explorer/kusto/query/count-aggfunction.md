---
title: count() (función de agregación) - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe count() (función de agregación) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/23/2020
ms.openlocfilehash: 59b898d44507d844db1f714ef15effa004e5546f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81517007"
---
# <a name="count-aggregation-function"></a>count() (función de agregación)

Devuelve un recuento de los registros por grupo de integración (o en total, si la integración se realiza sin agrupación).

* Sólo se puede utilizar en el contexto de la agregación dentro de [la suma](summarizeoperator.md)
* Utilice la función de agregación [countif](countif-aggfunction.md) `true`para contar solo los registros para los que se devuelve algún predicado.

**Sintaxis**

Resumir`count()`

**Devuelve**

Devuelve un recuento de los registros por grupo de integración (o en total, si la integración se realiza sin agrupación).