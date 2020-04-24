---
title: todynamic(), toobject() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe todynamic(), toobject() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 138b0f978df699817c5dc5c14bafc4c06a95afc7
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81506161"
---
# <a name="todynamic-toobject"></a>todynamic(), toobject()

Interpreta `string` un [valor JSON](https://json.org/) como y [`dynamic`](./scalar-data-types/dynamic.md)devuelve el valor como . 

Es superior al uso de la [función extractjson()](./extractjsonfunction.md) cuando necesita extraer más de un elemento de un objeto compuesto JSON.

Alias a [parse_json().](./parsejsonfunction.md)

**Sintaxis**

`todynamic(`*json*`)`json`toobject(`*json* 
json`)`

**Argumentos**

* *json*: Un documento JSON.

**Devuelve**

Un objeto de tipo `dynamic` especificado por *json*.

*Nota*: Prefiere usar [dynamic()](./scalar-data-types/dynamic.md) cuando sea posible.