---
title: El tipo de datos decimales - Explorador de datos de Azure ( Azure Data Explorer) Microsoft Docs
description: En este artículo se describe el tipo de datos decimal es en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 62fd745c804ad4a50652e09cd722c17a4a61daac
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509901"
---
# <a name="the-decimal-data-type"></a>El tipo de datos decimal

El `decimal` tipo de datos representa un número decimal de 128 bits de ancho.

Los literales `decimal` del tipo de datos tienen la misma representación que . NET's `System.Data.SqlTypes.SqlDecimal`.

`decimal(1.0)`, `decimal(0.1)`, `decimal(1e5)` y son todos `decimal`literales de tipo .

Hay varias formas literales especiales:
* `decimal(null)`: Es el [valor nulo](null-values.md).