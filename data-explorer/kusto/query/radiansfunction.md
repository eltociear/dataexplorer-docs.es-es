---
title: radians() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe radians() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 0dc04c5f9593b6bd5fc61f57d20819cf7d2a178c
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81510666"
---
# <a name="radians"></a>radians()

Convierte el valor del ángulo en grados en valor en radianes, utilizando la fórmula`radians = (PI / 180 ) * angle_in_degrees`

**Sintaxis**

`radians(`*Un*`)`

**Argumentos**

* *a*: Angulo en grados (un número real).

**Devuelve**

* El ángulo correspondiente en radianes para un ángulo especificado en grados. 

**Ejemplos**

```kusto
print radians0 = radians(90), radians1 = radians(180), radians2 = radians(360) 

```

|radians0|radianes1|radianes2|
|---|---|---|
|1.5707963267949|3.14159265358979|6.28318530717959|