---
title: 'Biblioteca de cliente de introducción de Kusto: procedimientos recomendados: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las prácticas recomendadas de la biblioteca de cliente de Kusto ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/16/2019
ms.openlocfilehash: 2369d352e316f1d9b49de71fb197507280598754
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617942"
---
# <a name="kusto-ingest-client-library---best-practices"></a>Biblioteca de cliente de introducción de Kusto: procedimientos recomendados

## <a name="choosing-the-right-ingestclient-flavor"></a>Elección del tipo de IngestClient adecuado

El uso de [KustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) es el modo de ingesta de datos nativo recomendado. Aquí se detallan los motivos:
* La ingesta directa no es posible durante el tiempo de inactividad del motor (por ejemplo, durante la implementación), mientras que en el modo de ingesta en cola las solicitudes se conservan en la cola de Azure y el servicio Administración de datos volverá a intentarlo según sea necesario.
* El servicio Administración de datos es responsable de no sobrecargar el motor con las solicitudes de ingesta. Reemplazar este control (por ejemplo, mediante la ingesta directa) puede afectar gravemente al rendimiento del motor, tanto la ingesta como la consulta.
* Administración de datos agrega varias solicitudes de ingesta para optimizar el tamaño de la partición inicial (extensión) que se va a crear.
* Obtener comentarios sobre cada ingesta es fácil de hacer.

## <a name="tracking-ingest-operation-status"></a>Estado de la operación de introducción de seguimiento

El seguimiento del estado de la [operación de introducción](kusto-ingest-client-status.md#tracking-ingestion-status-kustoqueuedingestclient) es una característica útil. Sin embargo, si se activa para que los informes se realicen correctamente, se puede hacer un abuso sencillo hasta el punto en que se deshabilitará el servicio.

> [!WARNING]
> Se debe evitar la activación de notificaciones positivas para cada solicitud de ingesta de flujos de datos de gran volumen, ya que esto supone una carga extrema en los recursos de xStore subyacentes. La carga adicional puede conducir a una mayor latencia de ingesta e incluso a la falta de capacidad de respuesta del clúster.

## <a name="optimizing-for-throughput"></a>Optimizar el rendimiento

* La ingesta funciona mejor si se realiza en fragmentos grandes. Consume los mínimos recursos, genera las particiones de datos optimizadas para COGS y produce los artefactos de datos que mejor funcionan. Por lo general, se recomienda que los clientes que ingestan datos con la biblioteca Kusto. ingesta o directamente en el motor de envíen datos en lotes de **100 MB a 1 GB (sin comprimir)**
* El límite superior es importante cuando se trabaja directamente con el motor de para ayudar a reducir la cantidad de memoria utilizada por el proceso de ingesta. 

> [!NOTE]
> Al utilizar la `KustoQueuedIngestClient` clase, los bloques de datos demasiado grandes se dividirán en fragmentos más pequeños, y estos fragmentos pequeños se agregarán, en cierto grado, antes de llegar al motor.

* El límite inferior del tamaño de los datos ingeridos también es importante, aunque es menos crítico. La ingesta de datos en lotes pequeños cada vez y después es perfecta, aunque es ligeramente más eficaz que el uso de lotes grandes. La `KustoQueuedIngestClient` clase también resuelve el problema de los clientes que necesitan ingerir grandes cantidades de datos y no pueden procesarlos por lotes en fragmentos grandes antes de enviarlos al motor.

## <a name="factors-impacting-ingestion-throughput"></a>Factores que afectan al rendimiento de la ingesta

Varios factores pueden afectar al rendimiento de la ingesta. Al planear la canalización de ingesta, asegúrese de evaluar los puntos siguientes, lo que puede tener implicaciones significativas en el coste de ventas.

| Factor a tener en cuenta |  Descifrado                                                                                               |
|--------------------------|-----------------------------------------------------------------------------------------------------------|
| Formato de datos              | CSV es el formato más rápido para la ingesta, JSON normalmente tardará 2x o 3 veces más en el mismo volumen de datos.|
| Ancho de tabla              | Asegúrese de que solo introduce datos que realmente necesita. Cuanto más amplio sea la tabla, más columnas deberán codificarse e indexarse y, por lo tanto, menor será el rendimiento. Puede controlar qué campos se ingestan proporcionando una asignación de ingesta.|
| Ubicación de los datos de origen     | Evite las lecturas entre regiones para acelerar la ingesta.                                                       |
| Cargar en el clúster      | Cuando un clúster experimenta una carga de consultas elevada, las ingesta tardarán más tiempo en completarse, lo que reduce el rendimiento.|
| Patrón de ingesta        | La ingesta es óptima cuando el clúster se atiende con lotes de hasta 1 GB (administrados mediante `KustoQueuedIngestClient`). |

## <a name="optimizing-for-cogs"></a>Optimizar para COGS

El uso de bibliotecas de cliente de Kusto para ingerir datos en Kusto sigue siendo la opción más barata y más sólida. Nos instamos a nuestros clientes a que revisen sus métodos de ingesta y aprovechen las ventajas de los precios Azure Storage que harán que las transacciones BLOB sean significativamente rentables.

Para un mejor control de los costos de ingesta de Azure Explorador de datos y para reducir la factura mensual, limite el número de fragmentos de datos ingeridos (archivos, blobs y secuencias).

* **Prefiera la ingesta de grandes fragmentos de datos (hasta 1 GB de datos sin comprimir)**. 
    Muchos equipos intentan lograr una baja latencia al introducir decenas de millones de fragmentos de datos pequeños, lo que es extremadamente ineficaz y muy costoso. Cualquier cantidad de procesamiento por lotes en el lado cliente sería útil. 
* Asegúrese de **proporcionar el cliente Kusto. ingeri con un tamaño de datos preciso y sin comprimir**.
    Si no lo hace, pueden producirse transacciones de almacenamiento adicionales.
* **Evite** el envío de datos para la ingesta con `FlushImmediately` la `true`marca establecida en o el envío de `ingest-by` / `drop-by` fragmentos pequeños con etiquetas definidas.
    El uso de estos métodos impide que el servicio agregue correctamente los datos durante la ingesta y produce transacciones de almacenamiento innecesarias después de la ingesta, lo que afecta al coste de ventas.
    
    Además, el uso excesivo de estas puede dar lugar a la ingesta o al rendimiento de las consultas en el clúster.
    
