---
title: Configuración de claves administradas por el cliente mediante la CLI de Azure
description: En este artículo se describe cómo configurar el cifrado de claves administradas por el cliente en sus datos en Azure Data Explorer mediante la CLI de Azure.
author: orspod
ms.author: orspodek
ms.reviewer: astauben
ms.service: data-explorer
ms.topic: conceptual
ms.date: 06/01/2020
ms.openlocfilehash: bc8ce7a3d85a64a2a19e4194e7c9e6ab23b0fef6
ms.sourcegitcommit: a7e040fc844098323aa1c00e254bcbcd41fe587f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2020
ms.locfileid: "84426534"
---
# <a name="configure-customer-managed-keys-using-azure-cli"></a>Configuración de claves administradas por el cliente mediante la CLI de Azure

> [!div class="op_single_selector"]
> * [Portal](customer-managed-keys-portal.md)
> * [C#](customer-managed-keys-csharp.md)
> * [Plantilla de Azure Resource Manager](customer-managed-keys-resource-manager.md)
> * [CLI](customer-managed-keys-cli.md)
> * [PowerShell](customer-managed-keys-powershell.md)

[!INCLUDE [data-explorer-configure-customer-managed-keys](includes/data-explorer-configure-customer-managed-keys.md)]

## <a name="enable-encryption-with-customer-managed-keys-using-azure-cli"></a>Habilitación del cifrado con claves administradas por el cliente mediante la CLI de Azure
En este artículo se muestra cómo habilitar el cifrado de claves administradas por el cliente de la CLI de Azure. De forma predeterminada, el cifrado de Azure Data Explorer usa claves que administra Microsoft. Configure la cuenta de Azure Data Explorer para usar las claves administradas por el cliente y especifique la clave para la asociación al clúster.

1. Ejecute el siguiente comandos para iniciar sesión en Azure:

    ```azurecli-interactive
    az login
    ```

1. Establezca la suscripción en el que está registrado el clúster. Reemplace *MyAzureSub* por el nombre de la suscripción de Azure que quiere usar.

    ```azurecli-interactive
    az account set --subscription MyAzureSub
    ```

1. Ejecute el comando siguiente para establecer la clave nueva.
    ```azurecli-interactive
    az kusto cluster update --cluster-name "mytestcluster" --resource-group "mytestrg" --key-vault-properties key-name="<key-name>" key-version="<key-version>" key-vault-uri="<key-vault-uri>"
    ```
1. Ejecute el siguiente comando y compruebe la propiedad "keyVaultProperties" para comprobar que el clúster se ha actualizado correctamente.

    ```azurecli-interactive
    az kusto cluster show --cluster-name "mytestcluster" --resource-group "mytestrg"
    ```

