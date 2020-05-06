---
title: 'Referencia de Kusto. ingesta: permisos de ingesta: Azure Explorador de datos'
description: En este artículo se describen los permisos de ingesta de referencia de Kusto. ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e60eb6642a66fac81ce373f8f4d62de4f7217a91
ms.sourcegitcommit: 061eac135a123174c85fe1afca4d4208c044c678
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/05/2020
ms.locfileid: "82799703"
---
# <a name="kustoingest-reference---ingestion-permissions"></a>Referencia de Kusto. ingesta: permisos de ingesta

En este artículo se explica qué permisos se deben configurar en el servicio `Native` para que el trabajo se ingesta.

## <a name="prerequisites"></a>Prerrequisitos

* Para ver y modificar la configuración de autorización en las bases de datos y los servicios de Kusto, vea [comandos de control de Kusto](../../management/security-roles.md).

## <a name="references"></a>Referencias

* Azure AD aplicaciones usadas como entidades de seguridad de ejemplo en los ejemplos siguientes.
    * Prueba de la aplicación AAD (2a904276-1234-5678-9012-66fc53add60b; microsoft.com)
    * Aplicación AAD de ingesta interna de Kusto (76263cdb-1234-5678-9012-545644e9c404; microsoft.com)

## <a name="ingestion-permission-mode-for-queued-ingestion"></a>Modo de permiso de ingesta para la ingesta en cola

El modo de permiso de ingesta se define en [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient). Este modo limita la dependencia del código de cliente en el servicio Azure Explorador de datos. La ingesta se realiza mediante la publicación de un mensaje de ingesta de Kusto en una cola de Azure. La cola, también conocida como servicio de ingesta, procede del servicio Azure Explorador de datos. El cliente de introducción creará los artefactos de almacenamiento intermedio usando los recursos asignados por el servicio Azure Explorador de datos.

En el diagrama se describe la interacción del cliente de ingesta en cola con Kusto.

:::image type="content" source="../images/queued-ingest.jpg" alt-text="en cola: ingesta":::

### <a name="permissions-on-the-engine-service"></a>Permisos en el servicio de motor

Para optar a la ingesta de `T1` datos en `DB1`la tabla de la base de datos, la entidad de seguridad que realiza la operación de introducción debe tener autorización.
Los niveles de permisos mínimos `Database Ingestor` necesarios `Table Ingestor` son y que pueden ingerir datos en todas las tablas existentes en una base de datos o en una tabla específica existente.
Si se requiere la creación de `Database User` la tabla, también se debe asignar un rol de acceso superior.


|Role                 |PrincipalType        |PrincipalDisplayName
|---------------------|---------------------|------------
|`Database Ingestor`  |Azure AD aplicación |`Test App (app id: 2a904276-1234-5678-9012-66fc53add60b)`
|`Table Ingestor`     |Azure AD aplicación |`Test App (app id: 2a904276-1234-5678-9012-66fc53add60b)`

>`Kusto Internal Ingestion AAD App (76263cdb-1234-5678-9012-545644e9c404)`entidad de seguridad, la aplicación de ingesta interna Kusto, está asignada a immutably al `Cluster Admin` rol. Por lo tanto, se autoriza a ingerir datos en cualquier tabla. Esto es lo que sucede en las canalizaciones de ingesta administradas por Kusto.

La concesión de los permisos necesarios `DB1` en la `T1` base de `Test App (2a904276-1234-5678-9012-66fc53add60b in AAD tenant microsoft.com)` datos o la tabla a aplicación de Azure ad tendría el siguiente aspecto:

```kusto
.add database DB1 ingestors ('aadapp=2a904276-1234-5678-9012-66fc53add60b;microsoft.com') 'Test AAD App'
.add table T1 ingestors ('aadapp=2a904276-1234-5678-9012-66fc53add60b;microsoft.com') 'Test AAD App'
```
