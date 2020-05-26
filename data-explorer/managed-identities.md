---
title: Configuración de identidades administradas para el clúster de Azure Data Explorer
description: Obtenga información sobre la configuración de identidades administradas para el clúster de Azure Data Explorer.
author: saguiitay
ms.author: itsagui
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/12/2020
ms.openlocfilehash: 523330f5ace4d9f2d652eccbd746b039d66df749
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83374296"
---
# <a name="configure-managed-identities-for-your-azure-data-explorer-cluster"></a>Configuración de identidades administradas para el clúster de Azure Data Explorer

Una [identidad administrada de Azure Active Directory](/azure/active-directory/managed-identities-azure-resources/overview) permite al clúster acceder fácilmente a otros recursos protegidos por AAD, como Azure Key Vault. La plataforma Azure administra la identidad y no es necesario que lleve a cabo el aprovisionamiento ni la rotación de los secretos. En este artículo se muestra cómo crear una identidad administrada para los clústeres de Azure Data Explorer. La configuración de identidad administrada solo se admite actualmente para [permitir claves administradas por el cliente en el clúster](security.md#customer-managed-keys-with-azure-key-vault).

> [!Note]
> Las identidades administradas de Azure Data Explorer no se comportarán según lo previsto si el clúster de Azure Data Explorer se migra entre suscripciones o inquilinos. La aplicación deberá obtener una nueva identidad, para lo cual puede [deshabilitar](#disable-a-system-assigned-identity) y [volver a habilitar](#add-a-system-assigned-identity) la característica. Las directivas de acceso y los recursos de nivel inferior también deberán actualizarse para utilizar la nueva identidad.

## <a name="add-a-system-assigned-identity"></a>Adición de una identidad asignada por el sistema
                                                                                                    
Asigne una identidad asignada por el sistema que esté asociada al clúster y que se eliminará si se elimina el clúster. Un clúster solo puede tener una identidad asignada por el sistema. Para crear un clúster con una identidad asignada por el sistema se requiere la configuración de una propiedad adicional en el clúster. La identidad asignada por el sistema se agrega mediante C#, plantillas de ARM o Azure Portal, tal y como se explica a continuación.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

### <a name="add-a-system-assigned-identity-using-the-azure-portal"></a>Adición de una identidad asignada por el sistema mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

#### <a name="new-azure-data-explorer-cluster"></a>Nuevo clúster de Azure Data Explorer

1. [Creación de un clúster de Azure Data Explorer](create-cluster-database-portal.md#create-a-cluster) 
1. En la pestaña **Seguridad** > **Identidad asignada por el sistema**, seleccione **Activado**. Para quitar la identidad asignada por el sistema, seleccione **Desactivado**.
2. Seleccione **Siguiente: Etiquetas >** o **Revisar y crear** para crear el clúster.

    ![Adición de una identidad asignada por el sistema al nuevo clúster](media/managed-identities/system-assigned-identity-new-cluster.png)

#### <a name="existing-azure-data-explorer-cluster"></a>Clúster de Azure Data Explorer existente

1. Abra un clúster de Azure Data Explorer existente.
1. Seleccione **Configuración** > **Identidad** en el panel izquierdo del portal.
1. En el panel **Identidad** > pestaña **Asignado por el sistema**:
   1. Mueva el control deslizante **Estado** a **Activado**.
   1. Seleccione **Guardar**.
   1. En la ventana emergente, seleccione **Sí**.

    ![Adición de una identidad asignada por el sistema](media/managed-identities/turn-system-assigned-identity-on.png)

1. Después de unos minutos, la pantalla muestra: 
  * **Id. de objeto**: se usa para las claves administradas por el cliente 
  * **Asignaciones de roles**: haga clic en el vínculo para asignar los roles pertinentes

    ![Identidad asignada por el sistema activada](media/managed-identities/system-assigned-identity-on.png)

# <a name="c"></a>[C#](#tab/c-sharp)

### <a name="add-a-system-assigned-identity-using-c"></a>Adición de una identidad asignada por el sistema mediante C#

#### <a name="prerequisites"></a>Prerrequisitos

Para configurar una identidad administrada mediante el cliente de C# de Azure Data Explorer:

* Instale el [paquete NuGet de Azure Data Explorer (Kusto)](https://www.nuget.org/packages/Microsoft.Azure.Management.Kusto/).
* Instale el [paquete NuGet Microsoft.IdentityModel.Clients.ActiveDirectory](https://www.nuget.org/packages/Microsoft.IdentityModel.Clients.ActiveDirectory/) para la autenticación.
* [Cree una aplicación de Azure AD](/azure/active-directory/develop/howto-create-service-principal-portal) y una entidad de servicio con acceso a los recursos. La asignación de roles se agrega en el ámbito de la suscripción y se obtienen los objetos `Directory (tenant) ID`, `Application ID` y `Client Secret`necesarios.

#### <a name="create-or-update-your-cluster"></a>Creación o actualización del clúster

1. Cree o actualice el clúster mediante la propiedad `Identity`:

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
    var location = "Central US";
    var skuName = "Standard_D13_v2";
    var tier = "Standard";
    var capacity = 5;
    var sku = new AzureSku(skuName, tier, capacity);
    var identity = new Identity(IdentityType.SystemAssigned);
    var cluster = new Cluster(location, sku, identity: identity);
    await kustoManagementClient.Clusters.CreateOrUpdateAsync(resourceGroupName, clusterName, cluster);
    ```
    
2. Ejecute el siguiente comando para comprobar si el clúster se creó o actualizó correctamente con una identidad:

    ```csharp
    kustoManagementClient.Clusters.Get(resourceGroupName, clusterName);
    ```

    Si el resultado contiene `ProvisioningState` con el valor `Succeeded`, el clúster se creó o actualizó y debe tener las siguientes propiedades:

    ```csharp
    var principalId = cluster.Identity.PrincipalId;
    var tenantId = cluster.Identity.TenantId;
    ```

`PrincipalId` y `TenantId` se reemplazan por GUID. La propiedad `TenantId` identifica el inquilino de AAD al que pertenece la identidad. `PrincipalId` es un identificador único para la nueva identidad del clúster. En AAD, la entidad de servicio tiene el mismo nombre que asignó a la instancia de App Service o Azure Functions.

# <a name="arm-template"></a>[Plantilla ARM](#tab/arm)

### <a name="add-a-system-assigned-identity-using-an-azure-resource-manager-template"></a>Adición de identidad asignada por el sistema mediante una plantilla de Azure Resource Manager

Se puede utilizar una plantilla de Azure Resource Manager para automatizar la implementación de los recursos de Azure. Para obtener información sobre la implementación en Azure Data Explorer, consulte [Creación de un clúster y una base de datos de Azure Data Explorer mediante una plantilla de Azure Resource Manager](create-cluster-database-resource-manager.md).

Al agregar el tipo asignado por el sistema se indica a Azure que debe crear y administrar la identidad del clúster. Se puede crear cualquier recurso de tipo `Microsoft.Kusto/clusters` con una identidad mediante la inclusión de la siguiente propiedad en la definición de recursos: 

```json
"identity": {
    "type": "SystemAssigned"
}    
```

Por ejemplo:

```json
{
    "apiVersion": "2019-09-07",
    "type": "Microsoft.Kusto/clusters",
    "name": "[variables('clusterName')]",
    "location": "[resourceGroup().location]",
    "identity": {
        "type": "SystemAssigned"
    },
    "properties": {
        "trustedExternalTenants": [],
        "virtualNetworkConfiguration": null,
        "optimizedAutoscale": null,
        "enableDiskEncryption": false,
        "enableStreamingIngest": false,
    }
}
```

Cuando se crea el clúster, tiene las siguientes propiedades adicionales:

```json
"identity": {
    "type": "SystemAssigned",
    "tenantId": "<TENANTID>",
    "principalId": "<PRINCIPALID>"
}
```

`<TENANTID>` y `<PRINCIPALID>` se reemplazan por GUID. La propiedad `TenantId` identifica el inquilino de AAD al que pertenece la identidad. `PrincipalId` es un identificador único para la nueva identidad del clúster. En AAD, la entidad de servicio tiene el mismo nombre que asignó a la instancia de App Service o Azure Functions.

---

## <a name="disable-a-system-assigned-identity"></a>Deshabilitación de una identidad asignada por el sistema

Al quitar una identidad asignada por el sistema también se eliminará de AAD. Las identidades asignadas por el sistema también se quitan automáticamente de AAD cuando se elimina el recurso del clúster. Para quitar una identidad asignada por el sistema se puede deshabilitar la característica.  La identidad asignada por el sistema se quita mediante C#, plantillas de ARM o Azure Portal, tal y como se explica a continuación.

# <a name="azure-portal"></a>[Azure Portal](#tab/portal)

### <a name="disable-a-system-assigned-identity-using-the-azure-portal"></a>Deshabilitación de una identidad asignada por el sistema mediante Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Seleccione **Configuración** > **Identidad** en el panel izquierdo del portal.
1. En el panel **Identidad** > pestaña **Asignado por el sistema**:
    1. Mueva el control deslizante **Estado** a **Desactivado**.
    1. Seleccione **Guardar**.
    1. En la ventana emergente, seleccione **Sí** para deshabilitar la identidad asignada por el sistema. El panel **Identidad** se revierte a la misma condición que antes de la adición de la identidad asignada por el sistema.

    ![Identidad asignada por el sistema desactivada](media/managed-identities/system-assigned-identity.png)

# <a name="c"></a>[C#](#tab/c-sharp)

### <a name="remove-a-system-assigned-identity-using-c"></a>Eliminación de una identidad asignada por el sistema mediante C#

Ejecute el comando siguiente para quitar la identidad asignada por el sistema:

```csharp
var identity = new Identity(IdentityType.None);
var cluster = new Cluster(location, sku, identity: identity);
await kustoManagementClient.Clusters.CreateOrUpdateAsync(resourceGroupName, clusterName, cluster);
```

# <a name="arm-template"></a>[Plantilla ARM](#tab/arm)

### <a name="remove-a-system-assigned-identity-using-an-azure-resource-manager-template"></a>Eliminación de una identidad asignada por el sistema mediante una plantilla de Azure Resource Manager

Ejecute el comando siguiente para quitar la identidad asignada por el sistema:

```json
"identity": {
    "type": "None"
}
```

---

## <a name="next-steps"></a>Pasos siguientes

* [Protección de clústeres de Azure Data Explorer en Azure](security.md)
* [Protección del clúster en Azure Data Explorer - Azure Portal](manage-cluster-security.md) mediante la habilitación del cifrado en reposo.
 * [Configuración de claves administradas por el cliente mediante C#](customer-managed-keys-csharp.md)
 * [Configuración de claves administradas por el cliente mediante la plantilla de Azure Resource Manager](customer-managed-keys-resource-manager.md)
