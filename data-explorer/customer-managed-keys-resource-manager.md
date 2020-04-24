---
title: Configuración de claves administradas por el cliente en Azure Data Explorer mediante la plantilla de Azure Resource Manager
description: En este artículo se describe cómo configurar el cifrado de claves administradas por el cliente en sus datos en Azure Data Explorer mediante la plantilla de Azure Resource Manager.
author: saguiitay
ms.author: itsagui
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 01/06/2020
ms.openlocfilehash: 1af7404a20c7246b50f80385f666da886f0cc96b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492836"
---
# <a name="configure-customer-managed-keys-using-the-azure-resource-manager-template"></a>Configuración de claves administradas por el cliente mediante la plantilla de Azure Resource Manager

> [!div class="op_single_selector"]
> * [Portal](customer-managed-keys-portal.md)
> * [C#](customer-managed-keys-csharp.md)
> * [Plantilla de Azure Resource Manager](customer-managed-keys-resource-manager.md)

[!INCLUDE [data-explorer-configure-customer-managed-keys](includes/data-explorer-configure-customer-managed-keys.md)]

[!INCLUDE [data-explorer-configure-customer-managed-keys part 2](includes/data-explorer-configure-customer-managed-keys-b.md)]

## <a name="configure-encryption-with-customer-managed-keys"></a>Configuración del cifrado con claves que administra el cliente

En esta sección, configurará claves administradas por el cliente mediante plantillas de Azure Resource Manager. De forma predeterminada, el cifrado de Azure Data Explorer usa claves que administra Microsoft. En este paso, configure la cuenta de Azure Data Explorer para usar las claves administradas por el cliente y especifique la clave para la asociación al clúster.

Puede implementar la plantilla de Azure Resource Manager mediante Azure Portal o PowerShell.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "clusterName": {
      "type": "string",
      "defaultValue": "[concat('kusto', uniqueString(resourceGroup().id))]",
      "metadata": {
        "description": "Name of the cluster to create"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]",
      "metadata": {
        "description": "Location for all resources."
      }
    }
  },
  "variables": {},
  "resources": [
    {
      "name": "[parameters('clusterName')]",
      "type": "Microsoft.Kusto/clusters",
      "sku": {
        "name": "Standard_D13_v2",
        "tier": "Standard",
        "capacity": 2
      },
      "apiVersion": "2019-09-07",
      "location": "[parameters('location')]",
      "properties": {
        "keyVaultProperties": {
          "keyVaultUri": "<keyVaultUri>",
          "keyName": "<keyName>",
          "keyVersion": "<keyVersion"
        }
      }
    }
  ]
}
```

## <a name="update-the-key-version"></a>Actualización de la versión de la clave

Al crear una nueva versión de una clave, tendrá que actualizar el clúster para que utilice la versión nueva. En primer lugar, llame a `Get-AzKeyVaultKey` para obtener la versión más reciente de la clave. A continuación, actualice las propiedades del almacén de claves del clúster para usar la nueva versión de la clave, como se muestra en [configurar el cifrado con claves administradas por el cliente](#configure-encryption-with-customer-managed-keys).

## <a name="next-steps"></a>Pasos siguientes

* [Protección de clústeres de Azure Data Explorer en Azure](security.md)
* [Configuración de identidades administradas para el clúster de Azure Data Explorer](managed-identities.md)
* [Protección del clúster en Azure Data Explorer - Azure Portal](manage-cluster-security.md) mediante la habilitación del cifrado en reposo.
* [Configuración de claves administradas por el cliente mediante C#](customer-managed-keys-csharp.md)

