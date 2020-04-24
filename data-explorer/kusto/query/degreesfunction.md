---
title: degrees() - Explorador de Azure Data Explorer ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe degrees() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 1365d6c6629f4f4769d7c4b62491eaec25e4ec59
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516038"
---
# <a name="degrees"></a>degrees()

Convierte el valor del ángulo en radianes en valor en grados, utilizando la fórmula`degrees = (180 / PI ) * angle_in_radians`

**Sintaxis**

`degrees(`*Un*`)`

**Argumentos**

* *a*: Angulo en radianes (un número real).

**Devuelve**

* El ángulo correspondiente en grados para un ángulo especificado en radianes. 

**Ejemplos**

```kusto
print degrees0 = degrees(pi()/4), degrees1 = degrees(pi()*1.5), degrees2 = degrees(0)

```

|degrees0|degrees1|degrees2|
|---|---|---|
|45|270|0|
