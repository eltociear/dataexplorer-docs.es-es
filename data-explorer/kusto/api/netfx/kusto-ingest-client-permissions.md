---
title: 'Permisos de Kusto. Introducción: Azure Explorador de datos'
description: En este artículo se describen los permisos de ingesta de Kusto. ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 2e88ba9af0b9563274e15eff8d1c1f6e997fb45c
ms.sourcegitcommit: e66c5f4b833b4f6269bb7bfa5695519fcb11d9fa
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2020
ms.locfileid: "83630010"
---
# <a name="kustoingest---ingestion-permissions"></a>Kusto. ingesta: permisos de ingesta 

En este artículo se explica qué permisos se deben configurar en el servicio para `Native` que el trabajo se ingesta.

## <a name="prerequisites"></a>Prerrequisitos
 
* Para ver y modificar la configuración de autorización en las bases de datos y los servicios de Kusto, vea [comandos de control de Kusto](../../management/security-roles.md).

* Las aplicaciones de Azure Active Directory (Azure AD) usadas como entidades de seguridad de ejemplo en los ejemplos siguientes:
    * Aplicación de Azure AD de prueba (2a904276-1234-5678-9012-66fc53add60b; microsoft.com)
    * Aplicación de Azure AD de ingesta interna de Kusto (76263cdb-1234-5678-9012-545644e9c404; microsoft.com)
 
## <a name="ingestion-permission-mode-for-queued-ingestion"></a>Modo de permiso de ingesta para la ingesta en cola

El modo de permiso de ingesta se define en [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient). Este modo limita la dependencia del código de cliente en el servicio Azure Explorador de datos. La ingesta se realiza mediante la publicación de un mensaje de ingesta de Kusto en una cola de Azure. La cola, también conocida como servicio de ingesta, procede del servicio Azure Explorador de datos. El cliente de introducción creará los artefactos de almacenamiento intermedio usando los recursos asignados por el servicio Azure Explorador de datos.

En el diagrama se describe la interacción del cliente de ingesta en cola con Kusto.

:::image type="content" source="../images/kusto-ingest-client-permissions/queued-ingest.png" alt-text="Ingesta en cola":::

### <a name="permissions-on-the-engine-service"></a>Permisos en el servicio de motor

Para optar a la ingesta de datos en la tabla `T1` de la base de datos `DB1` , la entidad de seguridad que realiza la operación de introducción debe tener autorización.
Los niveles de permisos mínimos necesarios son `Database Ingestor` y `Table Ingestor` que pueden ingerir datos en todas las tablas existentes en una base de datos o en una tabla específica existente.
Si se requiere la creación de `Database User` la tabla, también se debe asignar un rol de acceso superior.


|Rol                 |PrincipalType        |PrincipalDisplayName
|---------------------|---------------------|------------
|`Database Ingestor`  |Azure AD aplicación |`Test App (app id: 2a904276-1234-5678-9012-66fc53add60b)`
|`Table Ingestor`     |Azure AD aplicación |`Test App (app id: 2a904276-1234-5678-9012-66fc53add60b)`

>`Kusto Internal Ingestion Azure AD App (76263cdb-1234-5678-9012-545644e9c404)`entidad de seguridad, la aplicación de ingesta interna Kusto, está asignada a immutably al `Cluster Admin` rol. Por lo tanto, se autoriza a ingerir datos en cualquier tabla. Esto es lo que sucede en las canalizaciones de ingesta administradas por Kusto.

La concesión de los permisos necesarios en la base de datos `DB1` o la tabla `T1` a aplicación de Azure ad `Test App (2a904276-1234-5678-9012-66fc53add60b in Azure AD tenant microsoft.com)` tendría el siguiente aspecto:

```kusto
.add database DB1 ingestors ('aadapp=2a904276-1234-5678-9012-66fc53add60b;microsoft.com') 'Test Azure AD App'
.add table T1 ingestors ('aadapp=2a904276-1234-5678-9012-66fc53add60b;microsoft.com') 'Test Azure AD App'
```
 