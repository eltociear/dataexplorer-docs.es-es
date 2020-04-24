---
title: translate() - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe translate() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/11/2019
ms.openlocfilehash: f128ce31af7fc720ee81b7471d3d74616197b8d4
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81505719"
---
# <a name="translate"></a>translate()

Reemplaza un conjunto de caracteres ('searchList') por otro conjunto de caracteres ('replacementList') en una cadena determinada.
La función busca caracteres en el 'searchList' y los reemplaza por los caracteres correspondientes en 'replacementList'

**Sintaxis**

`translate(`*searchList* `,` *replacementLista* `,` *de texto*`)`

**Argumentos**

* *searchList*: La lista de caracteres que se deben reemplazar
* *replacementList*: La lista de caracteres que deben reemplazar los caracteres de 'searchList'
* *texto*: Una cadena para buscar

**Devuelve**

*texto* después de reemplazar todas las ocurrencias de caracteres en 'replacementList' con los caracteres correspondientes en 'searchList'

**Ejemplos**

|Entrada                                 |Output   |
|--------------------------------------|---------|
|`translate("abc", "x", "abc")`        |`"xxx"`  |
|`translate("abc", "", "ab")`          |`""`     |
|`translate("krasp", "otsku", "spark")`|`"kusto"`|