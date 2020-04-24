---
title: Incorporación de entidades de seguridad de clúster para Azure Data Explorer con C#
description: En este artículo, aprenderá a agregar entidades de seguridad de clúster para Azure Data Explorer con C#.
author: lucygoldbergmicrosoft
ms.author: lugoldbe
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 02/03/2020
ms.openlocfilehash: b677768b87419833968de48eca2dd32c4fdb17cb
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492796"
---
# <a name="add-cluster-principals-for-azure-data-explorer-by-using-c"></a>Incorporación de entidades de seguridad de clúster para Azure Data Explorer con C#

> [!div class="op_single_selector"]
> * [C#](cluster-principal-csharp.md)
> * [Python](cluster-principal-python.md)
> * [Plantilla de Azure Resource Manager](cluster-principal-resource-manager.md)

El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. En este artículo, agregará entidades de seguridad de clúster para Azure Data Explorer con C#.

## <a name="prerequisites"></a>Prerequisites

* Si no tiene Visual Studio 2019 instalado, puede descargar y usar la versión **gratis** de [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Asegúrese de que habilita **Desarrollo de Azure** durante la instalación de Visual Studio.
* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* [Cree un clúster](create-cluster-database-csharp.md).

## <a name="install-c-nuget"></a>Instalación de NuGet C#

* Instale [Microsoft.Azure.Management.kusto](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/).
* Instale [Microsoft.Rest.ClientRuntime.Azure.Authentication](https://www.nuget.org/packages/Microsoft.Rest.ClientRuntime.Azure.Authentication) para llevar a cabo la autenticación.

[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

## <a name="add-a-cluster-principal"></a>Incorporación de una entidad de seguridad de clúster

En el ejemplo siguiente, se explica cómo se agrega una entidad de seguridad de clúster mediante programación.

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
//The cluster that is created as part of the Prerequisites
var clusterName = "mykustocluster";
string principalAssignmentName = "clusterPrincipalAssignment1";
string principalId = "xxxxxxxx";//User email, application ID, or security group name
string role = "AllDatabasesAdmin";//AllDatabasesAdmin or AllDatabasesViewer
string tenantIdForPrincipal = tenantId;
string principalType = "App";//User, App, or Group

var clusterPrincipalAssignment = new ClusterPrincipalAssignment(principalId, role, principalType, tenantId: tenantIdForPrincipal);
await kustoManagementClient.ClusterPrincipalAssignments.CreateOrUpdateAsync(resourceGroupName, clusterName, principalAssignmentName, clusterPrincipalAssignment);
```

|**Configuración** | **Valor sugerido** | **Descripción del campo**|
|---|---|---|
| tenantId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | El identificador de inquilino. También conocido como identificador de directorio.|
| subscriptionId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | Identificador de suscripción que se usa para la creación de recursos.|
| clientId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | Identificador del cliente de la aplicación que puede acceder a los recursos del inquilino.|
| clientSecret | *xxxxxxxxxxxxxx* | Secreto del cliente de la aplicación que puede acceder a los recursos del inquilino. |
| resourceGroupName | *testrg* | Nombre del grupo de recursos que contiene el clúster.|
| clusterName | *mykustocluster* | Nombre del clúster.|
| principalAssignmentName | *clusterPrincipalAssignment1* | Nombre del recurso principal de clúster.|
| principalId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | El identificador de la entidad de seguridad, que puede ser el correo electrónico del usuario, el identificador de la aplicación o el nombre del grupo de seguridad.|
| rol | *AllDatabasesAdmin* | El rol de la entidad de seguridad de clúster, que puede ser ' "AllDatabasesAdmin" o "AllDatabasesViewer".|
| tenantIdForPrincipal | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | El identificador de inquilino de la entidad de seguridad.|
| principalType | *Aplicación* | El tipo de la entidad de seguridad, que puede ser "User" (Usuario), "App" (Aplicación) o "Group" (Grupo)|

## <a name="next-steps"></a>Pasos siguientes

* [Incorporación de las entidades de seguridad de base de datos](database-principal-csharp.md)
