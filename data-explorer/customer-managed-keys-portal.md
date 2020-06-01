---
title: Configuración de claves administradas por el cliente mediante Azure Portal
description: En este artículo se describe cómo configurar el cifrado de claves administradas por el cliente en sus datos en Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: itsagui
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/26/2020
ms.openlocfilehash: a75329c6aaad4fa31301104f9407ac36d25c3002
ms.sourcegitcommit: 9fe6e34ef3321390ee4e366819ebc9b132b3e03f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/01/2020
ms.locfileid: "84257899"
---
# <a name="configure-customer-managed-keys-using-the-azure-portal"></a>Configuración de claves administradas por el cliente mediante Azure Portal

> [!div class="op_single_selector"]
> * [Portal](customer-managed-keys-portal.md)
> * [C#](customer-managed-keys-csharp.md)
> * [Plantilla de Azure Resource Manager](customer-managed-keys-resource-manager.md)
> * [CLI](customer-managed-keys-cli.md)

[!INCLUDE [data-explorer-configure-customer-managed-keys](includes/data-explorer-configure-customer-managed-keys.md)]

## <a name="enable-encryption-with-customer-managed-keys-in-the-azure-portal"></a>Habilitación del cifrado con claves administradas por el cliente en Azure Portal

En este artículo se muestra cómo habilitar el cifrado de claves administradas por el cliente mediante Azure Portal. De forma predeterminada, el cifrado de Azure Data Explorer usa claves que administra Microsoft. Configure la cuenta de Azure Data Explorer para usar las claves administradas por el cliente y especifique la clave para la asociación al clúster.

1. En [Azure Portal](https://portal.azure.com/), vaya al recurso de [clúster de Azure Data Explorer](create-cluster-database-portal.md#create-a-cluster). 
1. Seleccione **Configuración** > **Cifrado** en el panel izquierdo del portal.
1. En el panel **Cifrado**, seleccione **Activar** para la opción **Clave administrada por el cliente**.
1. Haga clic en **Seleccionar clave**.

    ![Configuración de claves administradas por el cliente](media/customer-managed-keys-portal/cmk-encryption-setting.png)

1. En la ventana **Seleccione clave de Azure Key Vault**, seleccione un **almacén de claves** existente en la lista desplegable. Si selecciona **Crear nuevo** para [crear un nuevo almacén de claves](/azure/key-vault/quick-create-portal#create-a-vault), se le redirigirá a la pantalla **Crear almacén de claves**.

1. Seleccione **Clave**.
1. Seleccione **Versión**.
1. Haga clic en **Seleccionar**.

    ![Seleccionar clave en Azure Key Vault](media/customer-managed-keys-portal/cmk-key-vault.png)

1. En el panel **Cifrado** que ahora contiene la clave, seleccione **Guardar**. Cuando se complete la creación de CMK, se mostrará un mensaje que lo indique en **Notificaciones**.

    ![Guardar la clave administrada por el cliente](media/customer-managed-keys-portal/cmk-encryption-setting.png)

Al habilitar las claves administradas por el cliente para el clúster de Azure Data Explorer, creará una identidad asignada por el sistema para el clúster, si no existe ninguna. Además, proporcionará los permisos get, wrapKey y unwrapKey necesarios para el clúster de Azure Data Explorer en la instancia de Key Vault seleccionada y obtendrá las propiedades de Key Vault. 

> [!NOTE]
> Seleccione **Desactivar** para quitar la clave administrada por el cliente después de crearla.

## <a name="next-steps"></a>Pasos siguientes

* [Protección de clústeres de Azure Data Explorer en Azure](security.md)
* [Protección del clúster en Azure Data Explorer - Azure Portal](manage-cluster-security.md) mediante la habilitación del cifrado en reposo.
* [Configuración de claves administradas por el cliente mediante la plantilla de Azure Resource Manager](customer-managed-keys-resource-manager.md)
* [Configuración de claves administradas por el cliente mediante C#](customer-managed-keys-csharp.md)



