---
title: 'Coherencia de la consulta: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la coherencia de las consultas en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 01/20/2019
ms.openlocfilehash: b66540af2d2d4bef4571249474cd618e69eb2261
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523110"
---
# <a name="query-consistency"></a>Coherencia de consultas

Kusto admite dos modelos de coherencia de consulta: **fuerte** y **débil.**

Las consultas de alta coherencia (predeterminadas) tienen una garantía de "leer mis cambios". Un cliente que envía un comando de control y recibe un acuse de recibo positivo de que el comando se ha completado correctamente se garantiza que cualquier consulta inmediatamente posterior observará los resultados del comando.

Las consultas de forma débily coherente (deben habilitarse explícitamente por el cliente) no hacen la garantía. Los clientes que realizan consultas pueden observar cierta latencia (normalmente de 1 a 2 minutos) entre los cambios y las consultas que reflejan esos cambios.

La ventaja de las consultas de forma débilmente coherente es que reduce la carga en el nodo del clúster que controla los cambios de la base de datos. En general, se recomienda que los clientes prueben primero el modelo fuertemente consistente y cambien al uso de una consistencia débil si es absolutamente necesario.

El cambio a consultas débilmente coherentes se realiza estableciendo la `queryconsistency` propiedad al realizar una llamada a la API [rest](../api/rest/request.md). Los usuarios del cliente Kusto .NET también pueden establecerlo en la cadena de [conexión de Kusto](../api/connection-strings/kusto.md) o como una marca en las propiedades de solicitud de [cliente.](../api/netfx/request-properties.md)