---
title: bag_keys() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe bag_keys() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: a73798dff27bf9f4c7d554d97804aef6b49e1de2
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81518214"
---
# <a name="bag_keys"></a>bag_keys()

Enumera todas las claves raíz de un objeto de bolsa de propiedades dinámico.

`bag_keys(`*objeto dinámico*`)`

**Devuelve**

Una matriz de claves, el orden es indeterminado.

**Ejemplos**

|Expression|Se evalúa como|
|---|---|
|`bag_keys(dynamic({'a':'b', 'c':123}))` | `['a','c']`|
|`bag_keys(dynamic({'a':'b', 'c':{'d':123}})) `|`['a','c']`|
|`bag_keys(dynamic({'a':'b', 'c':[{'d':123}]})) `|`['a','c']`|
|`bag_keys(dynamic(null))`|`null`|
|`bag_keys(dynamic({}))`|`[]`|
|`bag_keys(dynamic('a'))`|`null`|
|`bag_keys(dynamic([]))  `|`null`|