---
title: Uso de ingesta de streaming para ingerir datos en Azure Data Explorer
description: Aprenda a ingerir (cargar) datos en Azure Data Explorer mediante la ingesta de streaming.
author: orspod
ms.author: orspodek
ms.reviewer: tzgitlin
ms.service: data-explorer
ms.topic: conceptual
ms.date: 08/30/2019
ms.openlocfilehash: c0373d2e380f1a9fb826d0e40ffcc0284f6db09a
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373780"
---
# <a name="streaming-ingestion-preview"></a>Ingesta de streaming (versión preliminar)

Utilice la ingesta de streaming cuando precise una latencia baja con un tiempo de ingesta de menos de 10 segundos de datos de volumen variado. Se usa para la optimización del procesamiento operativo de muchas tablas (de una o varias bases de datos), donde el flujo de datos a cada tabla es relativamente pequeño (pocos registros por segundo), pero el volumen de ingesta de datos global es alto (miles de registros por segundo). 

Use la ingesta en bloque en lugar de la ingesta de streaming cuando la cantidad de datos crezca más de 1 MB por segundo y tabla. Lea [Introducción a la ingesta de datos](ingest-data-overview.md) para conocer más sobre los distintos métodos de ingesta.

## <a name="prerequisites"></a>Prerequisites

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* Inicie sesión en la [interfaz de usuario web](https://dataexplorer.azure.com/).
* Cree un [clúster y una base de datos de Azure Data Explorer](create-cluster-database-portal.md).

## <a name="enable-streaming-ingestion-on-your-cluster"></a>Habilitación de la ingesta de streaming en el clúster

> [!WARNING]
> Revise las [limitaciones](#limitations) antes de habilitar la ingesta de streaming.

1. En Azure Portal, vaya al clúster de Azure Data Explorer. En **Configuración**, seleccione **Configuraciones**. 
1. En el panel **Configuraciones**, seleccione **Activado** para habilitar la **ingesta de streaming**.
1. Seleccione **Guardar**.
 
    ![Ingesta de streaming activada](media/ingest-data-streaming/streaming-ingestion-on.png)
 
1. En la [interfaz de usuario web](https://dataexplorer.azure.com/), defina la [directiva de ingesta de streaming](kusto/management/streamingingestionpolicy.md) en las tablas o las bases de datos que van a recibir los datos de streaming. 

    > [!NOTE]
    > * Si la directiva está definida en el nivel de base de datos, todas las tablas de la base de datos están habilitadas para la ingesta de streaming.
    > * La directiva aplicada solo puede hacer referencia a los datos recién ingeridos y no a otras tablas de la base de datos.

## <a name="use-streaming-ingestion-to-ingest-data-to-your-cluster"></a>Uso de la ingesta de streaming para ingerir datos en un clúster

Hay dos tipos de ingesta de streaming admitidos:


* [**Centro de eventos**](ingest-data-event-hub.md), que se usa como origen de datos.
* La **ingesta personalizada** requiere que escriba una aplicación que use una de las bibliotecas cliente de Azure Data Explorer. Consulte el [ejemplo de ingesta de streaming](https://github.com/Azure/azure-kusto-samples-dotnet/tree/master/client/StreamingIngestionSample) para obtener una aplicación de ejemplo.

### <a name="choose-the-appropriate-streaming-ingestion-type"></a>Selección del tipo de ingesta de streaming adecuado

|   |Centro de eventos  |Ingesta personalizada  |
|---------|---------|---------|
|Retraso de datos entre el inicio de la ingesta y los datos disponibles para la consulta   |    Retraso más largo     |   Retraso más corto      |
|Sobrecarga de desarrollo    |   Configuración rápida y sencilla, sin sobrecarga de desarrollo    |   Gran sobrecarga de desarrollo para que la aplicación administre los errores y garantice la coherencia de los datos     |

## <a name="disable-streaming-ingestion-on-your-cluster"></a>Deshabilitación de la ingesta de streaming en el clúster

> [!WARNING]
> La deshabilitación de la ingesta de streaming podría tardar unas horas.

1. Quite la [directiva de ingesta de streaming](kusto/management/streamingingestionpolicy.md) de todas las tablas y bases de datos pertinentes. La eliminación de la directiva de ingesta de streaming desencadena el movimiento de datos de ingesta de streaming desde el almacenamiento inicial hasta el almacenamiento permanente en el almacén de columnas (extensiones o particiones). El movimiento de datos puede durar entre unos segundos y algunas horas, según la cantidad de datos en el almacenamiento inicial y cómo el clúster haga uso de la CPU y la memoria.
1. En Azure Portal, vaya al clúster de Azure Data Explorer. En **Configuración**, seleccione **Configuraciones**. 
1. En el panel **Configuraciones**, seleccione **Desactivado** para deshabilitar la **ingesta de streaming**.
1. Seleccione **Guardar**.

    ![Ingesta de streaming desactivada](media/ingest-data-streaming/streaming-ingestion-off.png)

## <a name="limitations"></a>Limitaciones

* La ingesta de streaming no admite [cursores de base de datos](kusto/management/databasecursor.md) ni [asignación de datos](kusto/management/mappings.md). Solo se admite la asignación de datos [creada previamente](kusto/management/create-ingestion-mapping-command.md). 
* El rendimiento y la capacidad de la ingesta de streaming se escalan cuando aumentan los tamaños de las máquinas virtuales y los clústeres. Las ingestas simultáneas se limitan a seis ingestas por núcleo. Por ejemplo, en el caso de las SKU de 16 núcleos, como las D14 y L16, la carga máxima admitida es de 96 ingestas simultáneas. En el caso de las SKU de dos núcleos, como la D11, la carga máxima admitida es de 12 ingestas simultáneas.
* El límite de tamaño de datos por solicitud de ingesta es de 4 MB.
* Las actualizaciones de esquema, como la creación y modificación de tablas y asignaciones de ingesta, pueden tardar hasta cinco minutos en el servicio de ingesta de streaming.
* Cuando se habilita la ingesta de streaming en un clúster, incluso cuando los datos no se ingieren a través de streaming, se usa parte del disco SSD local de las máquinas del clúster para los datos de ingesta de streaming y se reduce el almacenamiento disponible para la caché activa.
* No se pueden establecer [etiquetas de extensión](kusto/management/extents-overview.md#extent-tagging) en los datos de ingesta de streaming.

## <a name="next-steps"></a>Pasos siguientes

* [Consulta de datos en Azure Data Explorer](web-query-data.md)
