---
title: Configuración de claves administradas por el cliente mediante C#
description: En este artículo se describe cómo configurar el cifrado de claves administradas por el cliente en sus datos en Azure Data Explorer.
author: saguiitay
ms.author: itsagui
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 01/06/2020
ms.openlocfilehash: 1d562939430aa5040dc2ab96ae59611d2685e057
ms.sourcegitcommit: a7e040fc844098323aa1c00e254bcbcd41fe587f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2020
ms.locfileid: "84426549"
---
# <a name="configure-customer-managed-keys-using-c"></a>Configuración de claves administradas por el cliente mediante C#

> [!div class="op_single_selector"]
> * [Portal](customer-managed-keys-portal.md)
> * [C#](customer-managed-keys-csharp.md)
> * [Plantilla de Azure Resource Manager](customer-managed-keys-resource-manager.md)
> * [CLI](customer-managed-keys-cli.md)
> * [PowerShell](customer-managed-keys-powershell.md)

[!INCLUDE [data-explorer-configure-customer-managed-keys](includes/data-explorer-configure-customer-managed-keys.md)]

[!INCLUDE [data-explorer-configure-customer-managed-keys part 2](includes/data-explorer-configure-customer-managed-keys-b.md)]

## <a name="configure-encryption-with-customer-managed-keys"></a>Configuración del cifrado con claves que administra el cliente

En esta sección se muestra cómo configurar el cifrado de claves administradas por el cliente mediante el cliente de C# de Azure Data Explorer. 

### <a name="prerequisites"></a>Requisitos previos

* Si no tiene Visual Studio 2019 instalado, puede descargar y usar la versión **gratis** de [Visual Studio 2019 Community Edition](https://www.visualstudio.com/downloads/). Asegúrese de que habilita **Desarrollo de Azure** durante la instalación de Visual Studio.

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

### <a name="install-c-nuget"></a>Instalación de NuGet C#

* Instale el [paquete NuGet de Azure Data Explorer (Kusto)](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/).

* Instale el [paquete NuGet Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/) para la autenticación.

### <a name="authentication"></a>Authentication

Para ejecutar los ejemplos de este artículo, [cree una aplicación de Azure AD](/azure/active-directory/develop/howto-create-service-principal-portal) y una entidad de servicio con acceso a los recursos. Puede agregar una asignación de roles en el ámbito de la suscripción y obtener los objetos `Directory (tenant) ID`, `Application ID` y `Client Secret`necesarios.

### <a name="configure-cluster"></a>Configuración del inicio de sesión del clúster

De forma predeterminada, el cifrado de Azure Data Explorer usa claves que administra Microsoft. Configure la cuenta de Azure Data Explorer para usar las claves administradas por el cliente y especifique la clave para la asociación al clúster.

1. Actualice el clúster mediante el siguiente código:

    ```csharp
    var tenantId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Directory (tenant) ID
    var clientId = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
    var clientSecret = "xxxxxxxxxxxxxx";//Client Secret
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
    var clusterName = "mykustocluster";
    var keyName = "myKey";
    var keyVersion = "5b52b20e8d8a42e6bd7527211ae32654";
    var keyVaultUri = "https://mykeyvault.vault.azure.net/";
    var keyVaultProperties = new KeyVaultProperties (keyName, keyVersion, keyVaultUri);
    var clusterUpdate = new ClusterUpdate(keyVaultProperties: keyVaultProperties);
    await kustoManagementClient.Clusters.UpdateAsync(resourceGroupName, clusterName, clusterUpdate);
    ```

1. Ejecute el siguiente comando para comprobar si el clúster se actualizó correctamente:

    ```csharp
    kustoManagementClient.Clusters.Get(resourceGroupName, clusterName);
    ```

    Si el resultado contiene `ProvisioningState` con el valor `Succeeded`, significa que el clúster se ha actualizado correctamente.

## <a name="update-the-key-version"></a>Actualización de la versión de la clave

Al crear una nueva versión de una clave, tendrá que actualizar el clúster para que utilice la versión nueva. En primer lugar, llame a `Get-AzKeyVaultKey` para obtener la versión más reciente de la clave. A continuación, actualice las propiedades del almacén de claves del clúster para usar la nueva versión de la clave, como se muestra en [Configuración del clúster](#configure-cluster).

## <a name="next-steps"></a>Pasos siguientes

* [Protección de clústeres de Azure Data Explorer en Azure](security.md)
* [Configuración de identidades administradas para el clúster de Azure Data Explorer](managed-identities.md)
* [Protección del clúster en Azure Data Explorer - Azure Portal](manage-cluster-security.md) mediante la habilitación del cifrado en reposo.
* [Configuración de claves administradas por el cliente mediante la plantilla de Azure Resource Manager](customer-managed-keys-resource-manager.md)


