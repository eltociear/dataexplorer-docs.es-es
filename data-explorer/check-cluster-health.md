---
title: Comprobar el estado de un clúster del Explorador de datos de Azure
description: En este artículo se describen los pasos para supervisar el estado del clúster de Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: a280d8869a3790444a97c38f792a3d9eeb6bde1d
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492740"
---
# <a name="check-the-health-of-an-azure-data-explorer-cluster"></a>Comprobar el estado de un clúster del Explorador de datos de Azure

Hay varios factores que afectan al estado de un clúster del Explorador de datos de Azure, incluidos la CPU, la memoria y el subsistema de disco. En este artículo se muestran algunos pasos básicos que puede seguir para valorar el estado de un clúster.

1. Inicie sesión en [https://dataexplorer.azure.com](https://dataexplorer.azure.com).

1. En el panel izquierdo, seleccione el clúster y ejecute el siguiente comando.

    ```Kusto
    .show diagnostics
    | project IsHealthy
    ```
    Un resultado de 1 indica que el estado es correcto; un resultado de 0 indica que el clúster está en mal estado.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya al clúster.

1. En **Supervisión**, seleccione primero **Métricas** y, a continuación, **Keep Alive**, tal y como se muestra en la siguiente imagen. Un resultado cercano a 1 indica que el estado del clúster es correcto.

    ![Métrica Keep Alive para clústeres](media/check-cluster-health/portal-metrics.png)

1. Es posible agregar otras métricas al gráfico. Seleccione el gráfico y, luego, **Agregar métrica**. Seleccione otra métrica; en este ejemplo se muestra **CPU**.

    ![Adición de métrica](media/check-cluster-health/add-metric.png)

1. Si necesita ayuda para diagnosticar problemas relacionados con el estado de un clúster, abra una solicitud de soporte técnico en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).