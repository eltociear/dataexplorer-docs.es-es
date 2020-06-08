---
title: 'Coherencia de consultas: Azure Explorador de datos'
description: En este artículo se describe la coherencia de las consultas en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/20/2019
ms.openlocfilehash: 8b4d1df4dc9a035764f9d50a4f9c4dcf452da67e
ms.sourcegitcommit: 188f89553b9d0230a8e7152fa1fce56c09ebb6d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2020
ms.locfileid: "84512272"
---
# <a name="query-consistency"></a>Coherencia de consultas

Kusto admite dos modelos de coherencia de consulta: **Strong** y **débil**.

Las consultas fuertemente coherentes (valor predeterminado) tienen una garantía de "lectura-mis-cambios". Si envía un comando de control y recibe confirmación de que el comando se ha completado correctamente, se garantizará cualquier consulta que se siga inmediatamente después de que observe los resultados del comando.

Las consultas débilmente coherentes que debe habilitar explícitamente el cliente, no tienen esa garantía. Los clientes que realizan consultas pueden observar cierta latencia (normalmente 1-2 minutos) entre los cambios y las consultas que reflejan esos cambios.

La ventaja de las consultas débilmente coherentes es que reduce la carga en el nodo de clúster que controla los cambios en la base de datos. En general, se recomienda probar primero el modelo fuertemente coherente. Cambie al uso de consultas débilmente coherentes únicamente si es necesario.

Para cambiar a consultas débilmente coherentes, establezca la `queryconsistency` propiedad al realizar una [llamada a la API de REST](../api/rest/request.md). Los usuarios del cliente .NET también pueden establecerlo en la [cadena de conexión Kusto](../api/connection-strings/kusto.md) o como una marca en las propiedades de la [solicitud de cliente](../api/netfx/request-properties.md).
