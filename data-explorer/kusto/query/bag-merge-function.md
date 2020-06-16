---
title: 'bag_merge (): Explorador de datos de Azure'
description: En este artículo se describe bag_merge () en Azure Explorador de datos.
services: data-explorer
author: elgevork
ms.author: elgevork
ms.reviewer: ''
ms.service: data-explorer
ms.topic: reference
ms.date: 06/14/2020
ms.openlocfilehash: f5ca4888b0c3aad976d78c10bbbfa0810483c75b
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84784606"
---
# <a name="bag_merge"></a>bag_merge ()

Combina los objetos de contenedor de propiedades dinámicos de entrada en un objeto de contenedor de propiedades dinámico.

**Sintaxis**

`bag_merge(`*objeto dinámico* `,` *objeto dinámico*`)`

**Argumentos**

* Los objetos dinámicos de entrada separados por comas. La función admite 2 a 64 objetos dinámicos de entrada.

**Devuelve**

Devuelve un `dynamic` contenedor de propiedades. Resultados de la combinación de todos los objetos de contenedor de propiedades de entrada.
Si aparece una clave en más de un objeto de entrada, se elegirá un valor arbitrario (fuera de los valores posibles para esta clave).

**Ejemplo**

Expresión:

`print result = bag_merge(dynamic({ 'A1':12, 'B1':2, 'C1':3}), dynamic({ 'A2':81, 'B2':82, 'A1':1}))`

Se evalúa como:

`{
  "A1": 12,
  "B1": 2,
  "C1": 3,
  "A2": 81,
  "B2": 82
}`
