---
title: treepath() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe treepath() en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 27a0edc61a1d2317427454aaf74531ec395d067d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505702"
---
# <a name="treepath"></a>treepath()

Enumera todas las expresiones de ruta que identifican hojas en un objeto dinámico.

`treepath(`*objeto dinámico*`)`

**Devuelve**

Una matriz de expresiones de ruta.

**Ejemplos**

|Expression|Se evalúa como|
|---|---|
|`treepath(parse_json('{"a":"b", "c":123}'))` | `["['a']","['c']"]`|
|`treepath(parse_json('{"prop1":[1,2,3,4], "prop2":"value2"}'))`|`["['prop1']","['prop1'][0]","['prop2']"]`|
|`treepath(parse_json('{"listProperty":[100,200,300,"abcde",{"x":"y"}]}'))`|`["['listProperty']","['listProperty'][0]","['listProperty'][0]['x']"]`|