---
title: Incorporación de entidades de seguridad de base de datos para Azure Data Explorer con C#
description: En este artículo, aprenderá a agregar entidades de seguridad de base de datos para Azure Data Explorer con C#.
author: lucygoldbergmicrosoft
ms.author: lugoldbe
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 02/03/2020
ms.openlocfilehash: 17107c1164b0ff84600776afd32feb4ef40256ff
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493360"
---
# <a name="add-database-principals-for-azure-data-explorer-by-using-c"></a>Incorporación de entidades de seguridad de base de datos para Azure Data Explorer con C#

> [!div class="op_single_selector"]
> * [C#](database-principal-csharp.md)
> * [Python](database-principal-python.md)
> * [Plantilla de Azure Resource Manager](database-principal-resource-manager.md)

El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. En este artículo, agregará entidades de seguridad de base de datos para Azure Data Explorer con C#.

## <a name="prerequisites"></a>Prerequisites

* Si no tiene Visual Studio 2019 instalado, puede descargar y usar la versión **gratis** de [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Asegúrese de que habilita **Desarrollo de Azure** durante la instalación de Visual Studio.
* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* [Cree un clúster y una base de datos](create-cluster-database-csharp.md).

## <a name="install-c-nuget"></a>Instalación de NuGet C#

* Instale [Microsoft.Azure.Management.kusto](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/).
* Instale [Microsoft.Rest.ClientRuntime.Azure.Authentication](https://www.nuget.org/packages/Microsoft.Rest.ClientRuntime.Azure.Authentication) para llevar a cabo la autenticación.

[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

## <a name="add-a-database-principal"></a>Incorporación de una entidad de seguridad de base de datos

En el ejemplo siguiente, se explica cómo se agrega una entidad de seguridad de base de datos mediante programación.

```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client Secret
var subscriptionId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";

var serviceCreds = await ApplicationTokenProvider.LoginSilentAsync(tenantId, clientId, clientSecret);
var kustoManagementClient = new KustoManagementClient(serviceCreds)
{
    SubscriptionId = subscriptionId
};

var resourceGroupName = "testrg";
//The cluster and database that are created as part of the Prerequisites
var clusterName = "mykustocluster";
var databaseName = "mykustodatabase";
string principalAssignmentName = "databasePrincipalAssignment1";
string principalId = "xxxxxxxx";//User email, application ID, or security group name
string role = "Admin";//Admin, Ingestor, Monitor, User, UnrestrictedViewers, Viewer
string tenantIdForPrincipal = tenantId;
string principalType = "App";//User, App, or Group

var databasePrincipalAssignment = new DatabasePrincipalAssignment(principalId, role, principalType, tenantId: tenantIdForPrincipal);
await kustoManagementClient.DatabasePrincipalAssignments.CreateOrUpdateAsync(resourceGroupName, clusterName, databaseName, principalAssignmentName, databasePrincipalAssignment);
```

|**Configuración** | **Valor sugerido** | **Descripción del campo**|
|---|---|---|
| tenantId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | El identificador de inquilino. También conocido como identificador de directorio.|
| subscriptionId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | Identificador de suscripción que se usa para la creación de recursos.|
| clientId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | Identificador del cliente de la aplicación que puede acceder a los recursos del inquilino.|
| clientSecret | *xxxxxxxxxxxxxx* | Secreto del cliente de la aplicación que puede acceder a los recursos del inquilino. |
| resourceGroupName | *testrg* | Nombre del grupo de recursos que contiene el clúster.|
| clusterName | *mykustocluster* | Nombre del clúster.|
| databaseName | *mykustodatabase* | Nombre de la base de datos.|
| principalAssignmentName | *databasePrincipalAssignment1* | Nombre del recurso principal de base de datos.|
| principalId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | El identificador de la entidad de seguridad, que puede ser el correo electrónico del usuario, el identificador de la aplicación o el nombre del grupo de seguridad.|
| rol | *Administrador* | El rol de la entidad de seguridad de base de datos, que puede ser "Admin" (Administrador), "Ingestor" (Agente de ingesta), "Monitor" (Monitor), "User" (Usuario), "UnrestrictedViewers" (Visores no restringidos) o "Viewer" (Visor).|
| tenantIdForPrincipal | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | El identificador de inquilino de la entidad de seguridad.|
| principalType | *Aplicación* | El tipo de la entidad de seguridad, que puede ser "User" (Usuario), "App" (Aplicación) o "Group" (Grupo)|

## <a name="next-steps"></a>Pasos siguientes

* [Ingesta de datos mediante la biblioteca de Node de Azure Data Explorer](node-ingest-data.md)
