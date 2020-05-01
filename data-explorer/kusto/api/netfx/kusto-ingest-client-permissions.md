---
title: 'Referencia de Kusto. ingesta: permisos de ingesta: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen los permisos de ingesta de referencia de Kusto. ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: ec3b91056a4cfc584c0d7168549864b2b50ef5ae
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617925"
---
# <a name="kustoingest-reference---ingestion-permissions"></a>Referencia de Kusto. ingesta: permisos de ingesta
En este artículo se explica qué permisos se deben configurar en el servicio `Native` para que el trabajo se ingesta.


## <a name="prerequisites"></a>Prerrequisitos
* Para ver y modificar la configuración de autorización en las bases de datos y los servicios de Kusto, vea [comandos de control de Kusto](../../management/security-roles.md) 

## <a name="references"></a>Referencias
* Las siguientes aplicaciones de AAD se usan como entidades de seguridad de ejemplo en los ejemplos siguientes:
    * Prueba de la aplicación AAD (2a904276-1234-5678-9012-66fc53add60b; microsoft.com)
    * Aplicación AAD de ingesta interna de Kusto (76263cdb-1234-5678-9012-545644e9c404; microsoft.com)

## <a name="ingestion-permission-model-for-queued-ingestion"></a>Modelo de permiso de ingesta para la ingesta en cola
Este modo, definido en [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient), limita la dependencia del código de cliente en el servicio Azure explorador de datos. La ingesta se realiza mediante la publicación de un mensaje de ingesta de Kusto en una cola de Azure. La cola se adquiere del servicio Azure Explorador de datos. It'ss también conocido como servicio de ingesta.  El cliente de introducción creará los artefactos de almacenamiento intermedio mediante los recursos asignados por el servicio Azure Explorador de datos.

En el diagrama siguiente se describe la interacción del cliente de ingesta en cola con Kusto:<BR>

:::image type="content" source="../images/queued-ingest.jpg" alt-text="en cola: ingesta":::

### <a name="permissions-on-the-engine-service"></a>Permisos en el servicio de motor
Para optar a la ingesta de `T1` datos en `DB1`la tabla de la base de datos, la entidad de seguridad que realiza la operación de introducción debe tener autorización.
Los niveles de permisos mínimos `Database Ingestor` necesarios `Table Ingestor` son y que pueden ingerir datos en todas las tablas existentes en una base de datos o en una tabla específica existente.
Si se requiere la creación de `Database User` la tabla, también se debe asignar un rol de acceso superior.


|Role |PrincipalType    |PrincipalDisplayName
|--------|------------|------------
|`Database Ingestor` |Aplicación AAD |`Test App (app id: 2a904276-1234-5678-9012-66fc53add60b)`
|`Table Ingestor` |Aplicación AAD |`Test App (app id: 2a904276-1234-5678-9012-66fc53add60b)`

>`Kusto Internal Ingestion AAD App (76263cdb-1234-5678-9012-545644e9c404)`entidad de seguridad, la aplicación de ingesta interna Kusto, está immutably `Cluster Admin` asignada al rol y, por tanto, está autorizada para ingerir datos en cualquier tabla. Esto es lo que sucede en las canalizaciones de ingesta administradas por Kusto.

La concesión de los permisos necesarios `DB1` en la `T1` base de datos `Test App (2a904276-1234-5678-9012-66fc53add60b in AAD tenant microsoft.com)` o la tabla a la aplicación de AAD tendría el siguiente aspecto:
```kusto
.add database DB1 ingestors ('aadapp=2a904276-1234-5678-9012-66fc53add60b;microsoft.com') 'Test AAD App'
.add table T1 ingestors ('aadapp=2a904276-1234-5678-9012-66fc53add60b;microsoft.com') 'Test AAD App'
```
