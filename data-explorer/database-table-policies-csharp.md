---
title: Creación de directivas mediante el SDK de C# de Azure Data Explorer
description: En este artículo, aprenderá a crear directivas con C#.
author: lucygoldbergmicrosoft
ms.author: lugoldbe
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/24/2019
ms.openlocfilehash: 0d02b5916cabc3645a3d61a9154ed31061c762aa
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373898"
---
# <a name="create-database-and-table-policies-for-azure-data-explorer-by-using-c"></a>Creación de directivas de bases de datos y tablas para Azure Data Explorer con C#

> [!div class="op_single_selector"]
> * [C#](database-table-policies-csharp.md)
> * [Python](database-table-policies-python.md)
>

El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. En este artículo, va a crear directivas de bases de datos y tablas para Azure Data Explorer con C#.

## <a name="prerequisites"></a>Prerrequisitos

* Visual Studio 2019. Si no tiene Visual Studio 2019, puede descargar y usar la versión *gratis* de [Visual Studio Community 2019](https://www.visualstudio.com/downloads/). Asegúrese de que habilita **Desarrollo de Azure** durante la instalación de Visual Studio.
* Suscripción a Azure. Si es necesario, puede crear una [cuenta de Azure gratuita](https://azure.microsoft.com/free/) antes de empezar.
* [Una base de datos y un clúster de prueba](create-cluster-database-csharp.md).
* [Una tabla de prueba](net-standard-ingest-data.md#create-a-table-on-your-test-cluster).

## <a name="install-c-nuget"></a>Instalación de NuGet C#

* Instale el [paquete NuGet de Azure Data Explorer (Kusto)](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/).
* Instale el [paquete NuGet Microsoft.Azure.Kusto.Data.NETStandard](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Data.NETStandard/). (Opcional, para cambiar las directivas de la tabla).
* Instale el [paquete NuGet Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/) para la autenticación.

## <a name="authentication"></a>Authentication
Para ejecutar los ejemplos en este artículo, es preciso una aplicación de Azure Active Directory (Azure AD) y una entidad de servicio con acceso a los recursos. Puede usar la misma aplicación de Azure AD para realizar la autenticación desde [una base de datos o un clúster de prueba](create-cluster-database-csharp.md#authentication). Si desea usar otra aplicación de Azure AD, consulte este artículo sobre la [creación de una aplicación de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal) para crear una aplicación de Azure AD gratuita y agregar la asignación de roles en el ámbito de la suscripción. En este artículo también se explica cómo obtener `Directory (tenant) ID`, `Application ID` y `Client secret`. Es posible que tenga que agregar la nueva aplicación de Azure AD como entidad de seguridad en la base de datos. Para obtener más información, consulte [Administración de permisos de base de datos del Explorador de datos de Azure](manage-database-permissions.md).

## <a name="alter-database-retention-policy"></a>Directiva de retención de modificación de la base de datos
Establece una directiva de retención con un período de eliminación temporal de 10 días.
    
```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client secret
var subscriptionId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";
var authenticationContext = new AuthenticationContext($"https://login.windows.net/{tenantId}");
var credential = new ClientCredential(clientId, clientSecret);
var result = await authenticationContext.AcquireTokenAsync(resource: "https://management.core.windows.net/", clientCredential: credential);

var credentials = new TokenCredentials(result.AccessToken, result.AccessTokenType);

var kustoManagementClient = new KustoManagementClient(credentials)
{
    SubscriptionId = subscriptionId
};

var resourceGroupName = "testrg";
//The cluster and database that are created as part of the prerequisites
var clusterName = "mykustocluster";
var databaseName = "mykustodatabase";
await kustoManagementClient.Databases.UpdateAsync(resourceGroupName, clusterName, databaseName, new DatabaseUpdate(softDeletePeriod: TimeSpan.FromDays(10)));
```

## <a name="alter-database-cache-policy"></a>Directiva de caché de modificación de la base de datos
Establece una directiva de caché para la base de datos. Los cinco días anteriores de datos estarán en el SSD del clúster.

```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client secret
var subscriptionId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";
var authenticationContext = new AuthenticationContext($"https://login.windows.net/{tenantId}");
var credential = new ClientCredential(clientId, clientSecret);
var result = await authenticationContext.AcquireTokenAsync(resource: "https://management.core.windows.net/", clientCredential: credential);

var credentials = new TokenCredentials(result.AccessToken, result.AccessTokenType);

var kustoManagementClient = new KustoManagementClient(credentials)
{
    SubscriptionId = subscriptionId
};

var resourceGroupName = "testrg";
//The cluster and database that are created as part of the prerequisites
var clusterName = "mykustocluster";
var databaseName = "mykustodatabase";
await kustoManagementClient.Databases.UpdateAsync(resourceGroupName, clusterName, databaseName, new DatabaseUpdate(hotCachePeriod: TimeSpan.FromDays(5)));
```

## <a name="alter-table-cache-policy"></a>Directiva de caché de modificación de la tabla
Establece una directiva de caché para la tabla. Los cinco días anteriores de datos estarán en el SSD del clúster.

```csharp
var kustoUri = "https://<ClusterName>.<Region>.kusto.windows.net:443/";
var databaseName = "<DatabaseName>";
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client secret
var tableName = "<TableName>"

var kustoConnectionStringBuilder =
    new KustoConnectionStringBuilder(kustoUri)
    {
        FederatedSecurity = true,
        InitialCatalog = databaseName,
        ApplicationClientId = clientId,
        ApplicationKey = clientSecret,
        Authority = tenantId
    };

using (var kustoClient = KustoClientFactory.CreateCslAdminProvider(kustoConnectionStringBuilder))
{
    //dataHotSpan and indexHotSpan should have the same value
    var hotSpan = TimeSpan.FromDays(5);
    var command1 = CslCommandGenerator.GenerateAlterTableCachingPolicyCommand(tableName: tableName,
                    dataHotSpan: hotSpan, indexHotSpan: hotSpan);

    kustoClient.ExecuteControlCommand(command);
}
```

## <a name="add-a-new-principal-for-the-database"></a>Incorporación de una nueva entidad de seguridad para la base de datos
Agrega una nueva aplicación de Azure AD como entidad de seguridad de administración de la base de datos.

```csharp
var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var clientSecret = "xxxxxxxxxxxxxx";//Client Secret
var clientIdToAdd = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
var subscriptionId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";
var authenticationContext = new AuthenticationContext($"https://login.windows.net/{tenantId}");
var credential = new ClientCredential(clientId, clientSecret);
var result = await authenticationContext.AcquireTokenAsync(resource: "https://management.core.windows.net/", clientCredential: credential);

var credentials = new TokenCredentials(result.AccessToken, result.AccessTokenType);

var kustoManagementClient = new KustoManagementClient(credentials)
{
    SubscriptionId = subscriptionId
};

var resourceGroupName = "testrg";
//The cluster and database that are created as part of the prerequisites
var clusterName = "mykustocluster";
var databaseName = "mykustodatabase";
await kustoManagementClient.Databases.AddPrincipalsAsync(resourceGroupName, clusterName, databaseName,
                new DatabasePrincipalListRequest()
                {
                    Value = new List<DatabasePrincipal>()
                    {
                        new DatabasePrincipal("Admin", "<database_principle_name>", "App", appId: clientIdToAdd, tenantName:tenantId)
                    }
                });
```
## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre las directivas de bases de datos y tablas](kusto/management/policies.md)
