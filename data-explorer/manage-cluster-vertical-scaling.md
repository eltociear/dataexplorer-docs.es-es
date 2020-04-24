---
title: Administración del escalado vertical del clúster para satisfacer la demanda en Azure Data Explorer
description: En este artículo se describen los pasos para escalar y reducir verticalmente un clúster de Azure Data Explorer en función de los cambios en la demanda.
author: radennis
ms.author: radennis
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 07/14/2019
ms.openlocfilehash: 95275598febae2b6b0355a7bc3e512490dae500d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492596"
---
# <a name="manage-cluster-vertical-scaling-scale-up-in-azure-data-explorer-to-accommodate-changing-demand"></a>Administración del escalado vertical de clústeres en Azure Data Explorer para ajustarse a los cambios en la demanda

Ajustar el tamaño de un clúster de forma adecuada es fundamental para el rendimiento del Explorador de datos de Azure. El tamaño de un clúster estático puede provocar una infrautilización o sobreutilización, y ninguna de estas situaciones es la ideal.

Dado que la demanda en un clúster no se puede predecir con precisión absoluta, un enfoque mejor es *escalar* el clúster mediante la adición y la eliminación de recursos de CPU y capacidad según los cambios en la demanda. 

Existen dos flujos de trabajo de escalado para un clúster de Azure Data Explorer:

* [Escalado horizontal](manage-cluster-horizontal-scaling.md), también denominado reducción horizontal.
* Escalado vertical, también denominado reducción vertical.

En este artículo se explica el flujo de trabajo del escalado vertical:

## <a name="configure-vertical-scaling"></a>Configurar el escalado vertical

1. En Azure Portal, vaya al recurso de clúster de Azure Data Explorer. En **Configuración**, seleccione **Escalar verticalmente**.

1. En la ventana **Escalar verticalmente**, verá una lista de las SKU disponibles para el clúster. Por ejemplo, en la ilustración siguiente, solo cuatro SKU están disponibles.

    ![Escalado vertical](media/manage-cluster-vertical-scaling/scale-up.png)

    Las SKU están deshabilitadas porque son la SKU actual o no están disponibles en la región donde se encuentra el clúster.

1. Para cambiar la SKU, seleccione una SKU nueva y haga clic en **Seleccionar**.

> [!NOTE]
> * El proceso de escalado vertical puede tardar unos minutos y, durante ese tiempo, se suspenderá el clúster. 
> * El escalado vertical puede perjudicar el rendimiento del clúster.
> * El precio es una estimación de los costos de servicio de Azure Data Explorer y las máquinas virtuales del clúster. No se incluyen otros costos. Consulte la página de [estimación de costos](https://dataexplorer.azure.com/AzureDataExplorerCostEstimator.html) de Azure Data Explorer para obtener una estimación y la [página de precios](https://azure.microsoft.com/pricing/details/data-explorer/) de Azure Data Explorer para obtener información completa sobre los precios.

Ha configurado el escalado vertical para el clúster de Azure Data Explorer. Agregue otra regla para el escalado horizontal. Si necesita ayuda por problemas relacionados con el escalado de un clúster, abra una solicitud de soporte técnico en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

## <a name="next-steps"></a>Pasos siguientes

* [Administración del escalado horizontal de clústeres](manage-cluster-horizontal-scaling.md) para escalar horizontalmente de forma dinámica el recuento de instancias en función de las métricas que especifique.

* Supervise el uso de recursos con este artículo: [Supervisión del rendimiento, el mantenimiento y el uso de Azure Data Explorer con métricas](using-metrics.md)

