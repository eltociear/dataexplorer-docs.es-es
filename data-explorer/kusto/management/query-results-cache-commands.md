---
title: 'Caché de resultados de la consulta: Azure Explorador de datos'
description: En este artículo se describe la memoria caché de resultados de consulta en Azure Explorador de datos.
services: data-explorer
author: amitof
ms.author: amitof
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: reference
ms.date: 06/16/2020
ms.openlocfilehash: 630266fdaaaac51037a99a6a5243db58a232ae48
ms.sourcegitcommit: a8575e80c65eab2a2118842e59f62aee0ff0e416
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/17/2020
ms.locfileid: "84943113"
---
# <a name="query-results-cache"></a>Caché de resultados de consulta

La caché de resultados de la consulta es una memoria caché dedicada para almacenar los resultados de la consulta. Para obtener más información, vea [caché de resultados](../query/query-results-cache.md)de la consulta.

**Comandos de caché de resultados de consulta**

Kusto proporciona dos comandos para la administración de la memoria caché y la observación:

* [`Show cache`](show-query-results-cache-command.md): Use este comando para mostrar las estadísticas expuestas por la memoria caché de resultados.

* [`Clear cache(rhs:string)`](clear-query-results-cache-command.md): Use este comando para borrar los resultados almacenados en caché.
