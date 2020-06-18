---
title: Borrar caché de resultados de consulta-Azure Explorador de datos
description: En este artículo se describe el comando de administración para borrar esquemas de base de datos en caché en Azure Explorador de datos.
services: data-explorer
author: amitof
ms.author: amitof
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: reference
ms.date: 06/16/2020
ms.openlocfilehash: 9ddb94e005e88855bbeddd7d3cf8ab537a42d413
ms.sourcegitcommit: a8575e80c65eab2a2118842e59f62aee0ff0e416
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2020
ms.locfileid: "84943126"
---
# <a name="clear-query-results-cache"></a>Borrar caché de resultados de consulta

Borre todos los [resultados de consulta almacenados en caché](../query/query-results-cache.md) realizados en la base de datos de contexto.

**Sintaxis**

`.clear` `database` `cache` `queryresults`

**Devuelve**

Este comando devuelve una tabla con las columnas siguientes:

|Columna    |Tipo    |Descripción
|---|---|---
|NodeId|`string`|Identificador del nodo de clúster.
|Entradas|`long`|Número de entradas borradas por el nodo.

**Ejemplo**

```kusto
.clear database cache queryresults
```

|NodeId|Entradas|
|---|---|
|Nodo1|42
|Nodo2|0
