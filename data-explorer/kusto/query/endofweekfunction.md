---
title: endofweek() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe endofweek() en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 83c080c60e34dbfdf19f7dde870621e34ded836d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515834"
---
# <a name="endofweek"></a>endofweek()

Devuelve el final de la semana que contiene la fecha, desplazada por un desplazamiento, si se proporciona.

El último día de la semana se considera un sábado.

**Sintaxis**

`endofweek(`*fecha* `,`[*offset*]`)`

**Argumentos**

* `date`: La fecha de entrada.
* `offset`: un número opcional de semanas de desplazamiento desde la fecha de entrada (entero, predeterminado - 0).

**Devuelve**

Una fecha y hora que representa el final de la semana para el valor de *fecha* dado, con el desplazamiento, si se especifica.

**Ejemplo**

```kusto
  range offset from -1 to 1 step 1
 | project weekEnd = endofweek(datetime(2017-01-01 10:10:17), offset)  

```

|Semana|
|---|
|2016-12-31 23:59:59.9999999|
|2017-01-07 23:59:59.9999999|
|2017-01-14 23:59:59.9999999|