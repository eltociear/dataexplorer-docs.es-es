---
author: orspod
ms.service: data-explorer
ms.topic: include
ms.date: 01/07/2020
ms.author: orspodek
ms.openlocfilehash: 5187f89a939daf45c0a5826e483de52cb7784103
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83383870"
---
Azure Data Explorer cifra todos los datos en una cuenta de almacenamiento en reposo. De manera predeterminada, los datos se cifran con claves administradas por Microsoft. Para tener un mayor control sobre las claves de cifrado, puede proporcionar claves administradas por el cliente para el cifrado de datos. 

Las claves administradas por el cliente se deben almacenar en una instancia de [Azure Key Vault](/azure/key-vault/key-vault-overview). Puede crear sus propias claves y almacenarlas en un almacén de claves, o puede usar una API de Azure Key Vault para generarlas. El clúster de Azure Data Explorer y el almacén de claves deben estar en la misma región, pero pueden estar en distintas suscripciones. Para obtener una explicación detallada sobre las claves administradas por el cliente, consulte [Claves administradas por el cliente con Azure Key Vault](/azure/storage/common/storage-service-encryption) 

En este artículo se muestra cómo configurar las claves administradas por el cliente.

## <a name="configure-azure-key-vault"></a>Configuración de Azure Key Vault

Para configurar las claves administradas por el cliente con Azure Data Explorer, debe [establecer dos propiedades en el almacén de claves](/azure/key-vault/key-vault-ovw-soft-delete): **Eliminación temporal** y **No purgar**. Estas propiedades no están habilitadas de forma predeterminada. Para habilitar estas propiedades, realice **Enabling soft-delete** (Habilitación de la eliminación temporal) y **Enabling Purge Protection** (Habilitación de la protección de purgas) en [PowerShell](/azure/key-vault/key-vault-soft-delete-powershell) o la [CLI de Azure](/azure/key-vault/key-vault-soft-delete-cli) en un almacén de claves nuevo o existente. Solo se admiten claves RSA de tamaño de 2048. Para obtener más información sobre las claves, consulte [Claves en Key Vault](/azure/key-vault/about-keys-secrets-and-certificates#key-vault-keys).

> [!NOTE]
> El cifrado de datos con claves administradas por el cliente no se admite en los [clústeres iniciales y seguidores](../follower.md).

## <a name="assign-an-identity-to-the-cluster"></a>Asignación de una identidad al clúster

Para habilitar claves administradas por el cliente para el clúster, primero asigne una identidad administrada que haya asignado el sistema al clúster. Usará esta identidad administrada para conceder los permisos del clúster para obtener acceso al almacén de claves. Para configurar identidades administradas asignadas por el sistema, consulte [Identidades administradas](../managed-identities.md).
