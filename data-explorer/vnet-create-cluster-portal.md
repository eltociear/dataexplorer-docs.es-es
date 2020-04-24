---
title: Creación de un clúster y una base de datos de Azure Data Explorer en la red virtual
description: En este artículo aprenderá a crear un clúster de Azure Data Explorer en la red virtual.
author: orspod
ms.author: orspodek
ms.reviewer: basaba
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/24/2020
ms.openlocfilehash: 99c32122fc48aaea7428fa559d7289713849f34e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81494016"
---
# <a name="create-an-azure-data-explorer-cluster-in-your-virtual-network"></a>Creación de un clúster de Azure Data Explorer en la red virtual

Azure Data Explorer admite la implementación de un clúster en una subred de la red virtual (VNet). Esta funcionalidad le permite obtener acceso al clúster de forma privada desde su red virtual de Azure o de forma local, obtener acceso a recursos como el centro de eventos o el almacenamiento de su red virtual, y restringir el tráfico entrante y saliente.

## <a name="prerequisites"></a>Prerrequisitos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="create-network-security-group-nsg"></a>Creación de un grupo de seguridad de red (NSG)

Los [grupos de seguridad de red (NSG)](/azure/virtual-network/security-overview) proporcionan la capacidad de controlar el acceso a la red dentro de una red virtual. Se puede acceder a Azure Data Explorer mediante dos puntos de conexión: HTTPs (443) y TDS (1433). Se deben configurar las reglas de NSG siguientes para permitir el acceso a estos puntos de conexión para la administración, la supervisión y el funcionamiento correcto del clúster.

Para crear el grupo de seguridad de red:

1. Seleccione el botón **+ Crear un recurso** de la esquina superior izquierda del portal.
1. Busque *Grupo de seguridad de red*.
1. En **Grupo de seguridad de red**, en la parte inferior de la pantalla, seleccione **Crear**.
1. En la ventana **Crear grupo de seguridad de red**, rellene la siguiente información.

   ![Creación de un formulario de NSG](media/vnet-create-cluster-portal/network-security-group.png)

    **Configuración** | **Valor sugerido** | **Descripción del campo**
    |---|---|---|
    | Subscription | Su suscripción | Seleccione la suscripción de Azure que quiere usar para el servidor.|
    | Resource group | Su grupo de recursos | Use un grupo de recursos existente o cree uno. |
    | Nombre | AzureDataExplorerNsg | Elija un nombre que identifique el grupo de seguridad de red (NSG) en el grupo de recursos.  |
    | Region | *Oeste de EE. UU.* | Seleccione la región que mejor se adapte a sus necesidades.
    | | | |

1. Seleccione **Revisar y crear** para revisar los detalles del clúster, y **Crear** para aprovisionarlo.

1. Cuando la implementación se complete, seleccione **Ir al grupo de recursos**.

    ![Ir al recurso](media/vnet-create-cluster-portal/notification-nsg.png)

1. En la pestaña **Reglas de seguridad de entrada**, seleccione **+ Agregar**.
1. En la ventana **Agregar regla de seguridad de entrada**, rellene la siguiente información.

    ![Creación de un formulario de reglas de entrada de NSG](media/vnet-create-cluster-portal/nsg-inbound-rule.png)

    **Configuración** | **Valor sugerido** 
    |---|---|
    | Source | ServiceTag
    | Etiqueta de servicio de origen | AzureDataExplorerManagement
    | Source port ranges | *
    | Destination | VirtualNetwork
    | Intervalos de puertos de destino | *
    | Protocolo | TCP
    | Acción | Allow
    | Priority | 100
    | Nombre | AllowAzureDataExplorerManagement
    | | |
    
1. Repita los dos pasos anteriores para todas las dependencias de entrada y de salida según las [dependencias de la implementación de VNet](/azure/data-explorer/vnet-deployment#dependencies-for-vnet-deployment). Como alternativa, las reglas de salida se pueden reemplazar por una única regla para permitir el uso de *Internet* para los puertos 443 y 80.
    
    Las reglas de NSG para las dependencias de entrada y salida deben tener este aspecto:

    ![Reglas de grupo de seguridad de red](media/vnet-create-cluster-portal/nsg-rules.png)

## <a name="create-public-ip-addresses"></a>Creación de direcciones IP públicas

Para crear las direcciones IP públicas de la consulta (motor):

1. Seleccione el botón **+ Crear un recurso** de la esquina superior izquierda del portal.
1. Busque *Grupo de seguridad de red*.
1. En **Dirección IP pública**, en la parte inferior de la pantalla, seleccione **Crear**.
1. En el panel **Crear dirección IP pública**, rellene la siguiente información.
   
   ![Creación de un formulario de IP pública](media/vnet-create-cluster-portal/public-ip-blade.png)

    **Configuración** | **Valor sugerido** | **Descripción del campo**
    |---|---|---|
    | Versión de la dirección IP | IPv4 | Seleccione la versión de la dirección IP. Solo se admite IPv4.|
    | SKU | Estándar | Se requiere la opción **Estándar** para el punto de conexión de URI de consultas (motor). |
    | Nombre | engine-pip | Elija un nombre que identifique la dirección IP pública en el grupo de recursos.
    | Subscription | Su suscripción | Seleccione la suscripción de Azure que quiere usar para la dirección IP pública.|
    | Resource group | Su grupo de recursos | Use un grupo de recursos existente o cree uno. |
    | Location | *Oeste de EE. UU.* | Seleccione la región que mejor se adapte a sus necesidades.
    | | | |

1. Seleccione **Crear** para crear la dirección IP pública.

1. Para crear la dirección IP pública de ingesta (administración de datos), siga las mismas instrucciones y seleccione 
    * **Sku**: Básica
    * **Asignación de dirección IP**: estática

## <a name="create-virtual-network-and-subnet"></a>Creación de una red virtual y una subred

Para crear la red y la subred virtuales:

1. Seleccione el botón **+ Crear un recurso** de la esquina superior izquierda del portal.
1. Busque *Red virtual*.
1. En **Red virtual**, en la parte inferior de la pantalla, seleccione **Crear**.
1. En la ventana **Crear red virtual**, rellene la siguiente información.

   ![Creación de un formulario de red virtual](media/vnet-create-cluster-portal/vnet-blade.png)

    **Configuración** | **Valor sugerido** | **Descripción del campo**
    |---|---|---|
    | Subscription | Su suscripción | Seleccione la suscripción de Azure que quiere usar para el servidor.|
    | Resource group | Su grupo de recursos | Use un grupo de recursos existente o cree uno. |
    | Nombre | AzureDataExplorerVnet | Elija un nombre que identifique la red virtual en el grupo de recursos.
    | Region | *Oeste de EE. UU.* | Seleccione la región que mejor se adapte a sus necesidades.
    | | | |

    > [!NOTE]
    > En el caso de las cargas de trabajo de producción, planee el tamaño de la subred según el [planeamiento del tamaño de la subred en la red virtual.](/azure/data-explorer/vnet-deployment#plan-subnet-size-in-your-vnet)

1. Seleccione **Revisar y crear** para revisar los detalles del clúster, y **Crear** para aprovisionarlo.

1. Cuando la implementación se complete, seleccione **Ir al grupo de recursos**.
1. Vaya a la hoja **Subredes** y seleccione la subred **predeterminada**.
    
    ![Hoja Subredes](media/vnet-create-cluster-portal/subnets.png)

1. En la ventana de la subred **predeterminada**:
    1. Seleccione el **grupo de seguridad de red** en el menú desplegable. 
    1. Seleccione el nombre del grupo de seguridad de red; en este caso, **AzureDataExplorerNsg**. 
    1. **Guardar**

    ![Configuración de la subred](media/vnet-create-cluster-portal/subnet-nsg.png)

## <a name="create-a-cluster"></a>Crear un clúster

Cree un clúster de Azure Data Explorer con un conjunto definido de recursos de proceso y de almacenamiento en un grupo de recursos de Azure, tal y como se describe en [Creación de un clúster](create-cluster-database-portal.md#create-a-cluster).

1. Antes de acabar de crear el clúster, en la ventana **Crear un clúster de Azure Data Explorer**, seleccione la pestaña **Red** para proporcionar los datos de la red virtual con los recursos creados en las pestañas anteriores:

   ![Creación de un formulario de red virtual de clúster](media/vnet-create-cluster-portal/create-cluster-form-vnet.png)

    **Configuración** | **Valor sugerido** | **Descripción del campo**
    |---|---|---|
    | Subscription | Su suscripción | Seleccione la suscripción de Azure que quiere usar para los recursos de red.|
    | Virtual Network | AzureDataExplorerVnet | Elija la red virtual que ha creado en los pasos anteriores.
    | Subnet | default | Elija la subred que ha creado en los pasos anteriores.
    | Consultar la IP pública | engine-pip | Elija la dirección IP pública de la consulta que ha creado en los pasos anteriores.
    | IP pública de ingesta de datos | dm-pip | Elija la dirección IP pública de ingesta que ha creado en los pasos anteriores.
    | | | |

1. Seleccione **Revisar y crear** para crear el clúster.
1. Cuando la implementación se complete, seleccione **Ir al grupo de recursos**.

Para implementar el clúster de Azure Data Explorer en la red virtual, use la plantilla de Azure Resource Manager [Implementación del clúster de Azure Data Explorer en la red virtual](https://azure.microsoft.com/resources/templates/101-kusto-vnet/).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Implementación de Azure Data Explorer en la red virtual](vnet-create-cluster-portal.md)