---
title: Referencia de Kusto.Ingest - Permisos de ingesta - Explorador de azure Data Explorer Microsoft Docs
description: En este artículo se describe Kusto.Ingest Reference - Ingestion Permissions in Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 198d562f880b74f6df4c6318f72213ee865f8f28
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81502897"
---
# <a name="kustoingest-reference---ingestion-permissions"></a>Referencia de Kusto.Ingest - Permisos de ingesta
En este artículo se explica qué permisos deben `Native` configurarse en el servicio para que la ingesta funcione.



## <a name="prerequisites"></a>Prerrequisitos
* Este artículo indica cómo utilizar los comandos de [control de Kusto](../../management/security-roles.md) para ver y modificar la configuración de autorización en los servicios y bases de datos de Kusto
* Las siguientes aplicaciones de AAD se utilizan como entidades principales de ejemplo en ejemplos a continuación:
    * Probar la aplicación AAD (2a904276-1234-5678-9012-66fc53add60b;microsoft.com)
    * Kusto Internal Ingestion AAD App (76263cdb-1234-5678-9012-545644e9c404;microsoft.com)

## <a name="ingestion-permission-model-for-queued-ingestion"></a>Modelo de permiso de ingesta para la ingesta en cola
Definido en [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient), este modo limita la dependencia del código de cliente en el servicio Kusto. La ingesta se realiza mediante la publicación de un mensaje de ingesta de Kusto en una cola de Azure, que, a su vez, se adquiere de Kusto Data Management (también. Ingestión). El cliente de ingesta creará cualquier artefacto de almacenamiento intermedio mediante los recursos asignados por el servicio de administración de datos de Kusto.<BR>

En el diagrama siguiente se describe la interacción del cliente de ingesta en cola con Kusto:<BR>

![texto alternativo](../images/queued-ingest.jpg "en cola-ingest")

### <a name="permissions-on-the-engine-service"></a>Permisos en el servicio de motor
Para calificar para la ingesta `T1` de `DB1` datos en la tabla en la base de datos, la entidad de seguridad que realiza la operación de ingesta debe estar autorizada para ello.
Los niveles mínimos de permisos requeridos son `Database Ingestor` y `Table Ingestor` pueden ingerigar datos en todas las tablas existentes en una base de datos o en una tabla existente específica, en consecuencia.
Si se requiere `Database User` la creación de tablas, o también se debe asignar un rol de acceso superior.


|Role |PrincipalType    |PrincipalDisplayName
|--------|------------|------------
|Base de datos *** Ingestor |Aplicación aAAD |Aplicación de prueba (identificador de aplicación: 2a904276-1234-5678-9012-66fc53add60b)
|Tabla *** Ingestor |Aplicación aAAD |Aplicación de prueba (identificador de aplicación: 2a904276-1234-5678-9012-66fc53add60b)

>`Kusto Internal Ingestion AAD App (76263cdb-1234-5678-9012-545644e9c404)`principal (Kusto internal Ingestion App) se `Cluster Admin` asigna inmutablemente al rol y, por lo tanto, se autoriza para ingerir datos a cualquier tabla (es decir, lo que está sucediendo en las canalizaciones de ingesta administradas por Kusto).

Conceder los permisos `DB1` necesarios en la base de datos o la tabla `T1` a la aplicación AAD `Test App (2a904276-1234-5678-9012-66fc53add60b in AAD tenant microsoft.com)` tendría el siguiente aspecto:
```kusto
.add database DB1 ingestors ('aadapp=2a904276-1234-5678-9012-66fc53add60b;microsoft.com') 'Test AAD App'
.add table T1 ingestors ('aadapp=2a904276-1234-5678-9012-66fc53add60b;microsoft.com') 'Test AAD App'
```

