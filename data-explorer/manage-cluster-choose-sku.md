---
title: Selección de la SKU de VM correcta para el clúster de Azure Data Explorer
description: En este artículo se describe cómo debe seleccionar el tamaño de SKU óptimo para el clúster de Azure Data Explorer.
author: avneraa
ms.author: avnera
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 07/14/2019
ms.openlocfilehash: 2d078f9715a0cfa171f0c88776a4ab78c15215a8
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81494488"
---
# <a name="select-the-correct-vm-sku-for-your-azure-data-explorer-cluster"></a>Selección de la SKU de máquina virtual correcta para el clúster de Azure Data Explorer 

Al crear un nuevo clúster u optimizar un clúster para una carga de trabajo cambiante, Azure Data Explorer ofrece varias SKU de máquina virtual entre las que elegir. Las máquinas virtuales se han elegido detenidamente para permitir el costo más óptimo posible para cualquier carga de trabajo. 

El servicio Azure Data Explorer administra completamente el tamaño y la SKU de la máquina virtual del clúster de administración de datos. Estos vienen determinados por factores como el tamaño de la máquina virtual del motor y la carga de trabajo de ingesta. 

Puede cambiar la SKU de la máquina virtual por el clúster del motor en cualquier momento. Para ello, es necesario [escalar verticalmente el clúster](manage-cluster-vertical-scaling.md). Es mejor empezar con el tamaño más pequeño de la SKU que se adapte al escenario inicial. Tenga en cuenta que el escalado vertical del clúster provoca un tiempo de inactividad de hasta 30 minutos mientras se vuelve a crear el clúster con la nueva SKU de máquina virtual.

> [!TIP]
> Las [RI (instancias reservadas)](https://docs.microsoft.com/azure/virtual-machines/windows/prepay-reserved-vm-instances) de proceso se pueden aplicar al clúster de Azure Data Explorer.  

En este artículo se describen las diferentes opciones de SKU de máquina virtual y se proporcionan los detalles técnicos que pueden ayudarle a tomar la mejor decisión.

## <a name="select-a-cluster-type"></a>Selección de un tipo de clúster

Azure Data Explorer ofrece dos tipos de clústeres:

* **Producción**: los clústeres de producción contienen dos nodos para los clústeres de administración de datos y del motor y operan en virtud del [Acuerdo de Nivel de Servicio](https://azure.microsoft.com/support/legal/sla/data-explorer/v1_0/) de Azure Data Explorer.

* **Desarrollo/pruebas (sin Acuerdo de Nivel de Servicio)** : los clústeres de Desarrollo/pruebas tienen un único nodo D11_v2 para el clúster de motor y un solo nodo D1 para el clúster de administración de datos. Este tipo de clúster es la configuración de menor costo debido al bajo recuento de instancias y a que no se aplica ningún recargo por marcado de motor. Para esta configuración de clúster, no hay ningún Acuerdo de Nivel de Servicio, ya que carece de redundancia.

## <a name="sku-types"></a>Tipos de SKU

Cuando cree un clúster de Azure Data Explorer, seleccione la SKU de máquina virtual *óptima* para la carga de trabajo planeada. Puede elegir entre las dos familias de SKU de Azure Data Explorer siguientes:

* **D v2**: la SKU D, que se ha optimizado para el proceso y se incluye en dos tipos:
    * La propia máquina virtual
    * La máquina virtual agrupada con discos de Premium Storage

* **LS**: la SKU L, que está optimizada para almacenamiento. Tiene un tamaño de SSD mucho mayor que la SKU D que tiene un precio similar.

Las diferencias principales entre los tipos de SKU disponibles se describen en la tabla siguiente:
 
| Atributo | SKU D | SKU L |
|---|---|---
|**SKU pequeñas**|El tamaño mínimo es D11 con dos núcleos|El tamaño mínimo es L4 con cuatro núcleos |
|**Disponibilidad**|Disponible en todas las regiones (la versión DS + PS tiene una disponibilidad más limitada)|Disponible en algunas regiones |
|**Costo por caché de&nbsp;GB por núcleo**|Alto con la SKU D, bajo con la versión DS + PS|Lo más bajo con la opción de pago por uso |
|**Precio de instancias reservadas (RI)**|Descuento alto (más del 55&nbsp;% para un compromiso de tres años)|Descuento menor (20&nbsp;% para un compromiso de tres años) |  

## <a name="select-your-cluster-vm"></a>Selección de la máquina virtual del clúster 

Para seleccionar la máquina virtual del clúster, [configure el escalado vertical](manage-cluster-vertical-scaling.md#configure-vertical-scaling). 

Con varias opciones de SKU de máquina virtual entre las que elegir, puede optimizar los costos de los requisitos frecuentes de rendimiento y caché para su escenario. 
* Si necesita un rendimiento óptimo para un volumen de consulta elevado, la SKU ideal debe optimizarse para el proceso. 
* Si necesita consultar grandes volúmenes de datos con una carga de consultas relativamente baja, la SKU optimizada para el almacenamiento puede ayudar a reducir los costos y seguir proporcionando un rendimiento excelente.

Como el número de instancias por clúster para las SKU pequeñas es limitado, es preferible usar máquinas virtuales más grandes que tengan más RAM. Más RAM es necesaria para algunos tipos de consulta que ponen más demanda en el recurso de RAM, como las consultas que utilizan `joins`. Por lo tanto, si se tiene en cuenta las opciones de escalado, es recomendable escalar verticalmente a una SKU más grande en lugar de escalar horizontalmente mediante la adición de más instancias.

## <a name="vm-options"></a>Opciones de máquina virtual

En la tabla siguiente se describen las especificaciones técnicas para las máquinas virtuales del clúster de Azure Data Explorer:

|**Nombre**| **Categoría** | **Tamaño de SSD** | **Núcleos** | **RAM** | **Discos de Premium Storage (1&nbsp;TB)**| **Número mínimo de instancias por clúster** | **Número máximo de instancias por clúster**
|---|---|---|---|---|---|---|---
|D11 v2| Optimizada para proceso | 75&nbsp;GB    | 2 | 14&nbsp;GB | 0 | 1 | 8 (excepto para la SKU de desarrollo/pruebas en que es 1)
|D12 v2| Optimizada para proceso | 150&nbsp;GB   | 4 | 28&nbsp;GB | 0 | 2 | 16
|D13 v2| Optimizada para proceso | 307&nbsp;GB   | 8 | 56&nbsp;GB | 0 | 2 | 1,000
|D14 v2| Optimizada para proceso | 614&nbsp;GB   | 16| 112&nbsp;GB | 0 | 2 | 1,000
|DS13 v2 + 1&nbsp;TB&nbsp;PS| Optimizado para el almacenamiento | 1&nbsp;TB | 8 | 56&nbsp;GB | 1 | 2 | 1,000
|DS13 v2 + 2&nbsp;TB&nbsp;PS| Optimizado para el almacenamiento | 2&nbsp;TB | 8 | 56&nbsp;GB | 2 | 2 | 1,000
|DS14 v2 + 3&nbsp;TB&nbsp;PS| Optimizado para el almacenamiento | 3&nbsp;TB | 16 | 112&nbsp;GB | 2 | 2 | 1,000
|DS14 v2 + 4&nbsp;TB&nbsp;PS| Optimizado para el almacenamiento | 4&nbsp;TB | 16 | 112&nbsp;GB | 4 | 2 | 1,000
|L4s v1| Optimizado para el almacenamiento | 650&nbsp;GB | 4 | 32&nbsp;GB | 0 | 2 | 16
|L8s v1| Optimizado para el almacenamiento | 1,3&nbsp;TB | 8 | 64&nbsp;GB | 0 | 2 | 1,000
|L16s_1| Optimizado para el almacenamiento | 2,6&nbsp;TB | 16| 128&nbsp;GB | 0 | 2 | 1,000

* Consulte la lista de SKU de máquinas virtuales actualizadas por región mediante [ListSkus API](/dotnet/api/microsoft.azure.management.kusto.clustersoperationsextensions.listskus?view=azure-dotnet) de Azure Data Explorer. 
* Más información sobre las [diferentes SKU](/azure/virtual-machines/windows/sizes). 

## <a name="next-steps"></a>Pasos siguientes

* Puede [escalar o reducir verticalmente](manage-cluster-vertical-scaling.md) el clúster del motor en cualquier momento mediante el cambio la SKU de la máquina virtual, en función de las distintas necesidades. 

* Puede [escalar o reducir horizontalmente](manage-cluster-horizontal-scaling.md) el tamaño del clúster del motor para modificar la capacidad, en función de las exigencias cambiantes.

