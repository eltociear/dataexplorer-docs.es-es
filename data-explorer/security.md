---
title: Protección de clústeres de Azure Data Explorer en Azure
description: Obtenga información sobre cómo proteger clústeres en Azure Data Explorer.
author: saguiitay
ms.author: itsagui
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 01/06/2020
ms.openlocfilehash: 786950011f10e25d6bcb72061212c1878e79d45a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492368"
---
# <a name="secure-azure-data-explorer-clusters-in-azure"></a>Protección de clústeres de Azure Data Explorer en Azure

Este artículo sirve de introducción a la seguridad en Azure Data Explorer para ayudar a proteger sus datos y recursos en la nube, así como a satisfacer las necesidades de seguridad de su negocio. Es importante proteger la seguridad de los clústeres. La protección de los clústeres incluye una o varias características de Azure, como las de almacenamiento y acceso seguro. En este artículo se proporciona información para ayudarle a proteger la seguridad del clúster.

## <a name="managed-identities-for-azure-resources"></a>Identidades administradas de recursos de Azure

Un desafío común al compilar aplicaciones en la nube es la administración de las credenciales del código para autenticar los servicios en la nube. Proteger las credenciales es una tarea esencial. Las credenciales no deben almacenarse en estaciones de trabajo del desarrollador ni registrarse en el control de código fuente. Azure Key Vault proporciona una manera segura de almacenar credenciales, secretos y otras claves pero el código tiene que autenticarse en Key Vault para recuperarlos.

Las identidades administradas de Azure Active Directory (Azure AD) para recursos de Azure resuelven este problema. Esta característica proporciona a los servicios de Azure una identidad de sistema administrada automáticamente en Azure AD. Puede usar esta identidad para autenticarse en cualquier servicio que admita la autenticación de Azure AD, incluido Key Vault, sin necesidad de credenciales en el código. Para obtener más información acerca de este servicio, consulte la página de información general [Identidades administradas de recursos de Azure](/azure/active-directory/managed-identities-azure-resources/overview).

## <a name="data-encryption"></a>Cifrado de datos

### <a name="azure-disk-encryption"></a>Azure Disk Encryption

[Azure Disk Encryption](/azure/security/azure-security-disk-encryption-overview) ayuda a custodiar y proteger sus datos con el fin de satisfacer los compromisos de cumplimiento y seguridad de su organización. Proporciona cifrado de volúmenes tanto para los discos de datos como para los del sistema operativo de las máquinas virtuales del clúster. Azure Disk Encryption también se integra con [Azure Key Vault](/azure/key-vault/), lo que nos permite controlar y administrar los secretos y las claves de cifrado de discos, y garantiza el cifrado de todos los datos de los discos de VM. 

### <a name="customer-managed-keys-with-azure-key-vault"></a>Claves administradas por el cliente con Azure Key Vault

De manera predeterminada, los datos se cifran con claves administradas por Microsoft. Para tener un mayor control sobre las claves de cifrado, puede proporcionar claves administradas por el cliente para el cifrado de datos. Puede administrar el cifrado de sus datos en el nivel de almacenamiento con sus propias claves. Una clave administrada por el cliente se usa para proteger y controlar el acceso a la clave de cifrado raíz, que se utiliza para cifrar y descifrar todos los datos. Las claves administradas por el cliente ofrecen más flexibilidad para crear, rotar, deshabilitar y revocar controles de acceso. También permite auditar las claves de cifrado que se usan para proteger los datos.

Utilice Azure Key Vault para almacenar las claves administradas por el cliente. Puede crear sus propias claves y almacenarlas en un almacén de claves, o puede usar una API de Azure Key Vault para generarlas. El clúster de Azure Data Explorer y Azure Key Vault deben estar en la misma región, pero pueden estar en distintas suscripciones. Para más información sobre Azure Key Vault, vea [¿Qué es Azure Key Vault?](/azure/key-vault/key-vault-overview). Para obtener una explicación detallada sobre las claves administradas por el cliente, consulte [Claves administradas por el cliente con Azure Key Vault](/azure/storage/common/storage-service-encryption). Configure claves administradas por el cliente en el clúster de Azure Data Explorer mediante [C#](/azure/data-explorer/customer-managed-keys-csharp) o la [plantilla de Azure Resource Manager](/azure/data-explorer/customer-managed-keys-resource-manager)

> [!Note]
> Las claves administradas por el cliente dependen de identidades administradas para los recursos de Azure, una característica de Azure Active Directory (Azure AD). Para configurar las claves administradas por el cliente en Azure Portal, debe configurar una identidad administrada **SystemAssigned** en el clúster, tal y como se detalla en [Configuración de identidades administradas para el clúster de Azure Data Explorer](/azure/data-explorer/managed-identities).

#### <a name="store-customer-managed-keys-in-azure-key-vault"></a>Almacenamiento de claves administradas por el cliente en Azure Key Vault

Para habilitar las claves administradas por el cliente en un clúster, utilice una instancia de Azure Key Vault para almacenar las claves. Debe habilitar las propiedades **Eliminación temporal** y **No purgar** en el almacén de claves. El almacén de claves debe estar en la misma suscripción que el clúster. Azure Data Explorer usa identidades administradas para que los recursos de Azure se autentiquen en el almacén de claves para las operaciones de cifrado y descifrado. Las identidades administradas no admiten escenarios entre directorios.

#### <a name="rotate-customer-managed-keys"></a>Rotación de claves administradas por el cliente

Las claves administradas por el cliente se pueden rotar en Azure Key Vault según las directivas de cumplimiento. Si se produce la rotación de la clave, hay que actualizar el clúster para usar el nuevo identificador URI de la clave. La rotación de la clave no desencadena un nuevo cifrado de los datos en el clúster. 

#### <a name="revoke-access-to-customer-managed-keys"></a>Revocación del acceso a las claves administradas por el cliente

Para revocar el acceso a las claves administradas por el cliente, use PowerShell o la CLI de Azure. Para más información, consulte la referencia de [PowerShell para Azure Key Vault](/powershell/module/az.keyvault/) o la referencia de la [CLI para Azure Key Vault](/cli/azure/keyvault). La revocación del acceso bloquea el acceso a todos los datos en el nivel de almacenamiento del clúster, ya que Azure Data Explorer no puede acceder a la clave de cifrado.

> [!Note]
> Si Azure Data Explorer identifica la revocación del acceso a una clave administrada por el cliente, suspenderá automáticamente el clúster para eliminar los datos almacenados en caché. Cuando se devuelve el acceso a la clave, el clúster debe reanudarse manualmente.

## <a name="role-based-access-control"></a>Control de acceso basado en rol

Mediante el [control de acceso basado en rol (RBAC)](/azure/role-based-access-control/overview), puede repartir las tareas entre el equipo y conceder solo el acceso necesario a los usuarios del clúster. En lugar de proporcionar a todos los empleados permisos no restringidos en el clúster, puede permitir solo determinadas acciones. Puede configurar el [control de acceso para las bases de datos](/azure/data-explorer/manage-database-permissions) en [Azure Portal](/azure/role-based-access-control/role-assignments-portal) mediante la [CLI de Azure](/azure/role-based-access-control/role-assignments-cli) o [Azure PowerShell](/azure/role-based-access-control/role-assignments-powershell).

## <a name="next-steps"></a>Pasos siguientes

* [Proteja su clúster en Azure Data Explorer - Portal](manage-cluster-security.md) mediante la habilitación del cifrado en reposo.
* [Configuración de identidades administradas para el clúster de Azure Data Explorer](managed-identities.md)
* [Configuración de claves administradas por el cliente mediante la plantilla de Azure Resource Manager](customer-managed-keys-resource-manager.md)
* [Configuración de claves administradas por el cliente mediante C#](customer-managed-keys-csharp.md)

