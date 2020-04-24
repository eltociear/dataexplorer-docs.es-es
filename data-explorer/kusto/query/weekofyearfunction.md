---
title: week_of_year() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe week_of_year() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/18/2020
ms.openlocfilehash: 1c3702165f01ab321f80bad900e59968092be2ed
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81504546"
---
# <a name="week_of_year"></a>week_of_year()

Devuelve un entero que representa el número de semana. El número de semana se calcula a partir de la primera semana de un año, que es la que incluye el primer jueves, según [ISO 8601.](https://en.wikipedia.org/wiki/ISO_8601#Week_dates)

```kusto
week_of_year(datetime("2015-12-14"))
```

**Sintaxis**

`week_of_year(`*a_date*`)`

**Argumentos**

* `a_date`: un valor `datetime`.

**Devuelve**

`week number`- El número de semana que contiene la fecha dada.

**Ejemplos**

|Entrada                                    |Output|
|-----------------------------------------|------|
|`week_of_year(datetime(2020-12-31))`     |`53`  |
|`week_of_year(datetime(2020-06-15))`     |`25`  |
|`week_of_year(datetime(1970-01-01))`     |`1`   |
|`week_of_year(datetime(2000-01-01))`     |`52`  |

> [!NOTE]
> `weekofyear()`es una variante obsoleta de esta función. `weekofyear()`no era compatible con la normas ISO 8601; la primera semana de un año se definió como la semana con el primer miércoles del año en ella.
La versión actual `week_of_year()`de esta función, , es compatible con ISO 8601; la primera semana de un año se define como la semana con el primer jueves del año en ella.