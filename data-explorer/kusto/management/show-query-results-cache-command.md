---
title: Mostrar caché de resultados de la consulta-Azure Explorador de datos
description: En este artículo se describe. Mostrar la memoria caché de resultados de consulta en Azure Explorador de datos.
services: data-explorer
author: amitof
ms.author: amitof
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: reference
ms.date: 06/16/2020
ms.openlocfilehash: bcbdb59355ce0461d735cbea902551c219479fd2
ms.sourcegitcommit: a8575e80c65eab2a2118842e59f62aee0ff0e416
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2020
ms.locfileid: "84943110"
---
# <a name="show-query-results-cache"></a>. Mostrar caché de resultados de consulta

Devuelve una tabla que muestra las estadísticas relacionadas con la [memoria caché de resultados](../query/query-results-cache.md)de la consulta.

**Sintaxis**

`.show` `query` `results` `cache`

**Salida**
 
|Parámetro de salida |Tipo |Descripción 
|---|---|---
|Aciertos  |long |Número de aciertos de caché.
|Errores  |long |El número de errores de caché.
|CacheCapacityInBytes |long |La capacidad de la memoria caché en bytes.
|UsedBytes  |long |Espacio utilizado en la memoria caché.
|Count  |long | El número de resultados de consulta únicos almacenados en la memoria caché.

**Limitaciones**

* La salida del comando solo refleja actualmente las estadísticas de caché recopiladas por el nodo en el que se ha descargado la solicitud.
* El comando solo muestra el historial "reciente".
