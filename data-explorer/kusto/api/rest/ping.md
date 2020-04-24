---
title: API REST de Kusto Ping - Explorador de Azure Data Explorer Microsoft Docs
description: En este artículo se describe la API de REST de Kusto Ping en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 14a3d3dd641ff435784eac4e813d98bac8c4a552
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81502591"
---
# <a name="kusto-ping-rest-api"></a>Kusto Ping REST API

La API REST de ping se proporciona para permitir que los dispositivos de red, como los equilibradores de carga, comprueben la capacidad de respuesta de red simple de los componentes front-end sin estado del clúster. Cuando se aplica un verbo GET a este punto de conexión, el clúster responde devolviendo una respuesta HTTP 200 OK. La API REST no está autenticada (el autor `Authorization` de la llamada no necesita enviar un encabezado HTTP).

- Ruta de acceso: `/v1/rest/ping`
- Verbo:`GET`
- Acciones: Responder `200 OK` con un mensaje
- Argumentos: Ninguno