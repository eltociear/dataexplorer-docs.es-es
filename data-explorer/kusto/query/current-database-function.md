---
title: current_database() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe current_database() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: c61717bbc8d202624b36088df5aed2ba3f3a8d2d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516752"
---
# <a name="current_database"></a>current_database()

Devuelve el nombre de la base de datos en el ámbito (base de datos en la que se resuelven todas las entidades de consulta si no se especifica ninguna otra base de datos).

**Sintaxis**

`current_database()`

**Devuelve**

El nombre de la base de `string`datos en el ámbito como un valor de tipo .

**Ejemplo**

```kusto
print strcat("Database in scope: ", current_database())
```