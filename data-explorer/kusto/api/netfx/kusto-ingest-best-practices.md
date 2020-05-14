---
title: 'Procedimientos recomendados de la biblioteca de cliente de Kusto ingesta: Azure Explorador de datos'
description: En este artículo se describen los procedimientos recomendados para la biblioteca de cliente de Kusto ingesta.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/16/2019
ms.openlocfilehash: d02d030a6deb468a4804c68965466e1917858be9
ms.sourcegitcommit: fd3bf300811243fc6ae47a309e24027d50f67d7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83382342"
---
# <a name="kusto-ingest-client-library---best-practices"></a>Biblioteca de cliente de introducción de Kusto: procedimientos recomendados

## <a name="select-the-right-ingestclient-flavor"></a>Seleccionar el tipo de IngestClient adecuado

Use [KustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient), es el modo de ingesta de datos nativo recomendado. El motivo es el siguiente:
* La ingesta directa no es posible durante el tiempo de inactividad del motor, como durante la implementación. En el modo de ingesta en cola, las solicitudes se conservan en la cola de Azure y el servicio Administración de datos volverá a intentarlo según sea necesario.
* El servicio Administración de datos impide que el motor se sobrecargue con solicitudes de ingesta. Reemplazar este control mediante la ingesta directa, por ejemplo, puede afectar gravemente a la ingesta del motor y al rendimiento de las consultas.
* Administración de datos agrega varias solicitudes de ingesta. La agregación optimiza el tamaño de la partición inicial (extensión) que se va a crear.
* Es fácil obtener comentarios sobre cada ingesta.

## <a name="avoid-tracking-ingest-operation-status"></a>Evitar el seguimiento del estado de la operación de introducción

El estado de la [operación de introducción de seguimiento](kusto-ingest-client-status.md#tracking-ingestion-status-kustoqueuedingestclient) es útil. Sin embargo, en el caso de los flujos de datos de gran volumen, se debe evitar la activación de notificaciones positivas para cada solicitud de ingesta. Este seguimiento pone una carga extrema en los recursos de xStore subyacentes que pueden conducir a una mayor latencia de ingesta e incluso a la falta de capacidad de respuesta del clúster.

## <a name="optimizing-for-throughput"></a>Optimizar el rendimiento

La ingesta funciona mejor si se realiza en fragmentos grandes. 
* Consume los mínimos recursos
* Genera las particiones de datos optimizadas para COGS y da como resultado las mejores transacciones de datos.

Se recomienda a los clientes que ingestan datos con la biblioteca Kusto. ingesta o directamente en el motor, que envíen datos en lotes de **100 MB** a **1 GB (sin comprimir)**
* El límite superior es importante cuando se trabaja directamente con el motor de para ayudar a reducir la cantidad de memoria utilizada por el proceso de ingesta. 

> [!NOTE]
> Al utilizar la `KustoQueuedIngestClient` clase, los bloques de datos demasiado grandes se dividirán en fragmentos más pequeños, y estos fragmentos pequeños se agregarán, en cierto grado, antes de llegar al motor.

* El límite inferior del tamaño de los datos ingeridos también es importante, aunque es menos crítico. La ingesta de datos en lotes pequeños cada vez y después es perfecta, aunque es ligeramente más eficaz que el uso de lotes grandes. La `KustoQueuedIngestClient` clase también resuelve el problema de los clientes que necesitan ingerir grandes cantidades de datos y no pueden procesarlos por lotes en fragmentos grandes antes de enviarlos al motor.

## <a name="other-factors-that-impact-ingestion-throughput"></a>Otros factores que afectan al rendimiento de la ingesta

Varios factores pueden afectar al rendimiento de la ingesta. Al planear la canalización de ingesta, asegúrese de evaluar los puntos siguientes, lo que puede tener implicaciones significativas en el coste de ventas.

| Factor a tener en cuenta |  Descripción                                                                                              |
|--------------------------|-----------------------------------------------------------------------------------------------------------|
| Formato de datos              | CSV es el formato más rápido para la ingesta. JSON normalmente tardará el 2x o el triple de tiempo en el mismo volumen de datos.|
| Ancho de tabla              | Asegúrese de que solo introduce datos que realmente necesita. Cuanto más amplio sea la tabla, más columnas deberán codificarse e indexarse, y menor será el rendimiento. Puede controlar qué campos se ingestan proporcionando una asignación de ingesta.       |
| Ubicación de los datos de origen     | Evite las lecturas entre regiones para acelerar la ingesta.                                                       |
| Cargar en el clúster      | Cuando un clúster experimenta una carga de consultas elevada, las ingesta tardarán más tiempo en completarse, lo que reduce el rendimiento.|

## <a name="optimizing-for-cogs"></a>Optimizar para COGS

El uso de bibliotecas de cliente de Kusto para ingerir datos en Azure Explorador de datos sigue siendo la opción más barata y más sólida. Nos instamos a nuestros clientes a que revisen sus métodos de ingesta y aprovechen las ventajas de los precios Azure Storage que harán que las transacciones BLOB sean significativamente rentables.

* **Limite el número de fragmentos de datos ingeridos**.
    Para un mejor control de los costos de ingesta de Azure Explorador de datos y para reducir la factura mensual, limite el número de fragmentos de datos ingeridos (archivos, blobs y secuencias).
* **Ingerir grandes fragmentos de datos (hasta 1 GB de datos sin comprimir)**. 
    Muchos equipos intentan lograr una baja latencia al introducir decenas de millones de fragmentos de datos pequeños, lo que es ineficaz y costoso. 
* **Procesamiento por lotes**. Cualquier cantidad de procesamiento por lotes en el lado cliente mejoraría la optimización. 
* **Proporcione el cliente Kusto. ingeri con un tamaño de datos exacto y sin comprimir**.
    Si no lo hace, pueden producirse transacciones de almacenamiento adicionales.
* **Evite** el envío de datos para la ingesta con la `FlushImmediately` marca establecida en `true` . Además, evite el envío de fragmentos pequeños con `ingest-by` / `drop-by` etiquetas definidas. Si usa estos métodos, se hará lo siguiente:
     * impedir que el servicio agregue correctamente los datos durante la ingesta
     * producir transacciones de almacenamiento innecesarias después de la ingesta
     * afectar a COGS
     * es probable que se produzca una ingesta o el rendimiento de las consultas en el clúster, si se usa de forma excesiva.
