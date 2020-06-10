---
title: Configuración de claves administradas por el cliente mediante PowerShell
description: En este artículo se describe cómo configurar el cifrado de claves administradas por el cliente en sus datos de Azure Data Explorer mediante PowerShell.
author: orspod
ms.author: orspodek
ms.reviewer: roshauli
ms.service: data-explorer
ms.topic: conceptual
ms.date: 06/04/2020
ms.openlocfilehash: ff9ad23e1e534e831e981e64cb2839f5e36d67c7
ms.sourcegitcommit: a7e040fc844098323aa1c00e254bcbcd41fe587f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2020
ms.locfileid: "84428351"
---
# <a name="configure-customer-managed-keys-using-powershell"></a>Configuración de claves administradas por el cliente mediante PowerShell

> [!div class="op_single_selector"]
> * [Portal](customer-managed-keys-portal.md)
> * [C#](customer-managed-keys-csharp.md)
> * [Plantilla de Azure Resource Manager](customer-managed-keys-resource-manager.md)
> * [CLI](customer-managed-keys-cli.md)
> * [PowerShell](customer-managed-keys-powershell.md)

[!INCLUDE [data-explorer-configure-customer-managed-keys](includes/data-explorer-configure-customer-managed-keys.md)]

## <a name="enable-encryption-with-customer-managed-keys-using-powershell"></a>Habilitación del cifrado con claves administradas por el cliente mediante PowerShell

En este artículo se muestra cómo habilitar el cifrado de claves administradas por el cliente mediante PowerShell. De forma predeterminada, el cifrado de Azure Data Explorer usa claves que administra Microsoft. Configure la cuenta de Azure Data Explorer para usar las claves administradas por el cliente y especifique la clave para la asociación al clúster.

1. Ejecute el siguiente comandos para iniciar sesión en Azure:

    ```azurepowershell-interactive
    Connect-AzAccount
    ```

1. Establezca la suscripción en el que está registrado el clúster.

    ```azurepowershell-interactive
    Set-AzContext -SubscriptionId "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    ```

1. Ejecute el comando siguiente para establecer la clave nueva.

    ```azurepowershell-interactive
    Update-AzKustoCluster -ResourceGroupName "mytestrg" -Name "mytestcluster" -KeyVaultPropertyKeyName "<key-name>" -KeyVaultPropertyKeyVaultUri "<key-vault-uri>" -KeyVaultPropertyKeyVersion "<key-version>"
    ```

1. Ejecute el siguiente comando y compruebe las propiedades "KeyVaultProperty..." para comprobar que el clúster se ha actualizado correctamente.

    ```azurepowershell-interactive
    Get-AzKustoCluster -Name "mytestcluster" -ResourceGroupName "mytestrg" | Format-List
    ```
