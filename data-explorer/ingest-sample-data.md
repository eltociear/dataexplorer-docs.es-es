---
title: Ingesta de datos de ejemplo en el Explorador de datos de Azure
description: Obtenga información sobre cómo ingerir (cargar) datos de ejemplo relacionadas con el tiempo en el Explorador de datos de Azure.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: quickstart
ms.date: 08/12/2019
ms.openlocfilehash: 3ece5a9d225e48654a0a3a96c3b7b78327565841
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81491920"
---
# <a name="quickstart-ingest-sample-data-into-azure-data-explorer"></a>Inicio rápido: Ingesta de datos de ejemplo en el Explorador de datos de Azure

En este artículo, se muestra cómo ingerir (cargar) datos de ejemplo en una base de datos del Explorador de datos de Azure. Hay [varias maneras de ingerir datos](ingest-data-overview.md); este artículo se centra en un enfoque básico que es adecuado para realizar pruebas.

> [!NOTE]
> Si ha completado [Ingesta de datos mediante la biblioteca de Python de Azure Data Explorer](python-ingest-data.md), ya tiene estos datos.

## <a name="prerequisites"></a>Prerrequisitos

[Base de datos y clúster de prueba](create-cluster-database-portal.md)

## <a name="ingest-data"></a>Ingerir datos

El conjunto de datos de ejemplo de **StormEvents** contiene datos relacionados con el tiempo de los [National Centers for Environmental Information](https://www.ncdc.noaa.gov/stormevents/).

1. Inicie sesión en [https://dataexplorer.azure.com](https://dataexplorer.azure.com).

1. En la parte superior izquierda de la aplicación, haga clic en **Agregar clúster**.

1. En el cuadro de diálogo **Agregar clúster**, escriba la URL del clúster con el formato `https://<ClusterName>.<Region>.kusto.windows.net/` y después haga clic en **Agregar**.

1. Pegue el siguiente comando y seleccione **Ejecutar** para crear una tabla de StormEvents.

    ```Kusto
    .create table StormEvents (StartTime: datetime, EndTime: datetime, EpisodeId: int, EventId: int, State: string, EventType: string, InjuriesDirect: int, InjuriesIndirect: int, DeathsDirect: int, DeathsIndirect: int, DamageProperty: int, DamageCrops: int, Source: string, BeginLocation: string, EndLocation: string, BeginLat: real, BeginLon: real, EndLat: real, EndLon: real, EpisodeNarrative: string, EventNarrative: string, StormSummary: dynamic)
    ```
1. Pegue el siguiente comando y seleccione **Ejecutar** para ingerir datos en la tabla de StormEvents.

    ```Kusto
    .ingest into table StormEvents h'https://kustosamplefiles.blob.core.windows.net/samplefiles/StormEvents.csv?st=2018-08-31T22%3A02%3A25Z&se=2020-09-01T22%3A02%3A00Z&sp=r&sv=2018-03-28&sr=b&sig=LQIbomcKI8Ooz425hWtjeq6d61uEaq21UVX7YrM61N4%3D' with (ignoreFirstRecord=true)
    ```

1. Una vez finalizada la ingesta, pegue la siguiente consulta, seleccione la consulta en la ventana y haga clic en **Ejecutar**.

    ```Kusto
    StormEvents
    | sort by StartTime desc
    | take 10
    ```
    La consulta devuelve los siguientes resultados de los datos de ejemplo ingeridos.

    ![Resultados de la consulta](media/ingest-sample-data/query-results.png)

## <a name="next-steps"></a>Pasos siguientes

* [Ingesta de datos de Azure Data Explorer](ingest-data-overview.md) para más información sobre los métodos de ingesta.
* [Inicio rápido: Consulta de datos en la interfaz de usuario web de Azure Data Explorer](web-query-data.md).
* [Escriba consultas](write-queries.md) con el lenguaje de consulta Kusto.
