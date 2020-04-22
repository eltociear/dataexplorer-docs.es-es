---
title: 'API REST de Azure Data Explorer: Azure Data Explorer | Microsoft Docs'
description: En este artículo se describen la API REST de Azure Data Explorer en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 05/29/2019
ms.openlocfilehash: 000561f96bd4174b94cca8e9db4c932a3cf3f0c6
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81490691"
---
# <a name="azure-data-explorer-rest-api"></a>API REST de Azure Data Explorer

En este artículo se describe cómo interactuar con Kusto a través de HTTPS.

## <a name="what-actions-are-supported"></a>¿Qué acciones se admiten?

La lista de acciones que admite un punto de conexión varía en función de que el punto de conexión sea del motor o de la administración de datos.

|Acción         |Verbo HTTP  |Plantilla de URI             |Motor|Administración de datos|¿Autenticación?|
|---------------|-----------|-------------------------|------|---------------|---------------|
|Consultar          |GET o POST|`/v1/rest/query`         |Sí   |No             |Sí            |
|Consultar          |GET o POST|`/v2/rest/query`         |Sí   |No             |Sí            |
|Administración     |POST       |`/v1/rest/mgmt`          |Sí   |Sí            |Sí            |
|UI             |GET        |`/`                      |Sí   |No             |No             |
|UI             |GET        |`/{dbname}`              |Sí   |No             |No             |

Donde **Acción** representa un grupo de actividades relacionadas:

* La acción **Consultar** envía una consulta al servicio y devuelve los resultados de la consulta.
* La acción **Administración** envía un comando de control al servicio y devuelve los resultados del comando de control.
* La acción **UI** se puede usar para iniciar un cliente de escritorio o un cliente web (a través de una respuesta de redirección http) para interactuar con el servicio.

Consulte [solicitud HTTP de consulta/administración](./request.md), [respuesta HTTP de consulta/administración](./response.md) y [respuesta HTTP de consulta v2](./response2.md) para más información acerca de la solicitud/respuesta HTTP de la consulta y las acciones de administración. Para obtener los detalles de la acción UI, consulte [UI (vínculo profundo](./deeplink.md)).