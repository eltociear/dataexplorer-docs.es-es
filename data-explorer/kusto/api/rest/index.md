---
title: 'API REST de Azure Data Explorer: Azure Data Explorer'
description: En este artículo se describen la API REST de Azure Data Explorer en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 05/29/2019
ms.openlocfilehash: 6abfe15490e5c633e1cb7912f4794ee90bec1947
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373602"
---
# <a name="azure-data-explorer-rest-api"></a>API REST de Azure Data Explorer

En este artículo se describe cómo interactuar con Kusto a través de HTTPS.

## <a name="supported-actions"></a>Acciones admitidas

La lista de acciones que admite un punto de conexión varía en función de que este sea del motor o de la administración de datos.

|Acción         |Verbo HTTP   |Plantilla de URI           |Motor|Administración de datos|Authentication |
|---------------|------------|-----------------------|------|---------------|---------------|
|Consultar          |GET o POST |/v1/rest/query         |Sí   |No             |Sí            |
|Consultar          |GET o POST |/v2/rest/query         |Sí   |No             |Sí            |
|Administración     |POST        |/v1/rest/mgmt          |Sí   |Sí            |Sí            |
|UI             |GET         |/                      |Sí   |No             |No             |
|UI             |GET         |/{dbname}              |Sí   |No             |No             |

Donde *Acción* representa un grupo de actividades relacionadas

* La acción Consultar envía una consulta al servicio y recibe los resultados de la consulta.
* La acción Administración envía un comando de control al servicio y recibe los resultados del comando de control.
* La acción Interfaz de usuario se puede usar para iniciar un cliente de escritorio o un cliente web. La acción se realiza a través de una respuesta de redirección HTTP para interactuar con el servicio.

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la solicitud HTTP y la respuesta de las acciones de consulta y administración, consulte:
 * [Solicitud HTTP de consulta/administración](./request.md)
 * [Respuesta HTTP de consulta/administración](./response.md)
 * [Respuesta HTTP de consulta v2](./response2.md)

Para más información sobre la acción Interfaz de usuario, consulte:
 * [Vínculo profundo de la interfaz de usuario](./deeplink.md)
