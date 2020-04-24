---
title: Protección de clústeres en Azure Data Explorer
description: En este artículo se describe cómo proteger un clúster en Azure Data Explorer dentro de Azure Portal.
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: conceptual
ms.date: 08/20/2019
ms.openlocfilehash: 0f935999b68a7283c032d43c42d688b273d5c450
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492600"
---
# <a name="secure-your-cluster-in-azure-data-explorer---azure-portal"></a>Protección de clústeres en Azure Data Explorer - Azure Portal

[Azure Disk Encryption](/azure/security/azure-security-disk-encryption-overview) ayuda a custodiar y proteger sus datos con el fin de satisfacer los compromisos de cumplimiento y seguridad de su organización. Proporciona cifrado de volúmenes tanto para los discos de datos como para los del sistema operativo de las máquinas virtuales del clúster. También se integra con [Azure Key Vault](/azure/key-vault/), lo que nos permite controlar y administrar los secretos y las claves de cifrado de discos, y garantiza el cifrado de todos los datos de los discos de VM. 
  
## <a name="enable-encryption-at-rest-in-the-azure-portal"></a>Habilitación del cifrado en reposo en Azure Portal
  
La configuración de seguridad del clúster le permite habilitar el cifrado de disco en el clúster. La habilitación del [cifrado en reposo](/azure/security/fundamentals/encryption-atrest) en su clúster proporciona protección de datos para los datos almacenados (en reposo). 

1. En Azure Portal, vaya al recurso de clúster de Azure Data Explorer. En el encabezado **Configuración**, seleccione **Seguridad**. 

    ![Activación del cifrado en reposo](media/manage-cluster-security/security-encryption-at-rest.png)

1. En la ventana **Seguridad**, seleccione **Activado** en el valor de seguridad de **Cifrado de disco**. 

1. Seleccione **Guardar**.
 
> [!NOTE]
> Seleccione **Desactivado** para deshabilitar el cifrado una vez que se ha habilitado.

## <a name="next-steps"></a>Pasos siguientes

[Comprobación del estado del clúster](/azure/data-explorer/check-cluster-health)
