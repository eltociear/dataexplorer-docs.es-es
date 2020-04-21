---
title: Biblioteca de cliente de ingesta de Kusto - Prácticas recomendadas - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe Kusto Ingest Client Library - Best Practices in Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 08/16/2019
ms.openlocfilehash: e28ec3f99abe4163b83721d753da98c0035d1e46
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523705"
---
# <a name="kusto-ingest-client-library---best-practices"></a>Kusto Ingest Client Library - Prácticas recomendadas

## <a name="choosing-the-right-ingestclient-flavor"></a>Elegir el sabor ingestClient adecuado
El uso de [KustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) es el modo de ingesta de datos nativo recomendado. Aquí se detallan los motivos:
* La ingesta directa es imposible durante el tiempo de inactividad del motor de Kusto (por ejemplo, durante la implementación), mientras que en el modo de ingesta en cola las solicitudes se conservan en la cola de Azure y el servicio de administración de datos volverá a intentarlo según sea necesario.
* El servicio de administración de datos es responsable de no sobrecargar el motor con solicitudes de ingesta. Reemplazar este control (por ejemplo, mediante la ingestión directa) podría afectar gravemente al rendimiento del motor, tanto la ingesta como la consulta.
* Administración de datos agrega varias solicitudes de ingesta para optimizar el tamaño de la partición inicial (extensión) que se va a crear.
* Hay una manera conveniente de obtener comentarios sobre cada ingesta, ya sea que tenga éxito o no.

## <a name="tracking-ingest-operation-status"></a>Seguimiento del estado de la operación de ingesta
El seguimiento del estado de la operación de [ingesta](kusto-ingest-client-status.md#tracking-ingestion-status-kustoqueuedingestclient) es una característica útil en Kusto, sin embargo, activarlo para la notificación de éxito se puede abusar fácilmente en la medida en que paralizará su servicio.<BR>

> [!WARNING]
> Debe evitarse activar las notificaciones positivas para cada solicitud de ingesta de flujos de datos de gran volumen, ya que esto pone una carga extrema en los recursos xStore subyacentes, > lo que podría dar lugar a una mayor latencia de ingesta e incluso a una falta de respuesta completa del clúster.

## <a name="optimizing-for-throughput"></a>Optimización para el rendimiento
* La ingesta funciona mejor (es decir, consume el menor tamaño de recursos durante la ingesta, genera la mayoría de las particiones de datos optimizadas para COGS y da como resultado los artefactos de datos de mejor rendimiento) si se realiza en fragmentos grandes. Por lo general, recomendamos a los clientes que ingien datos con la biblioteca Kusto.Ingest o directamente en el motor que envíen datos en lotes de **100 MB a 1 GB (sin comprimir)**
* El límite superior es importante cuando se trabaja directamente con el motor De Kusto `KustoQueuedIngestClient` para ayudar a reducir la cantidad de memoria utilizada por el proceso de ingesta (cuando se utiliza la clase, bloques de datos demasiado grandes se dividirán en el cliente en porciones más pequeñas, y pequeños fragmentos se agregarán hasta cierto grado, antes de llegar al motor de Kusto)
* El límite inferior del tamaño de los datos ingeridos también es importante, aunque menos crítico. La ingesta de datos en lotes pequeños de vez en cuando está perfectamente bien, aunque un poco menos eficiente que el uso de lotes grandes. `KustoQueuedIngestClient`clase también resuelve el problema para los clientes que necesitan ingerir grandes cantidades de datos y no pueden procesarlos por lotes en grandes trozos antes de enviarlos a Kusto

## <a name="factors-impacting-ingestion-throughput"></a>Factores que afectan el rendimiento de la ingesta
Varios factores pueden afectar al rendimiento de la ingesta. Al planear la canalización de ingesta de Kusto, asegúrese de evaluar los siguientes puntos, que pueden tener implicaciones significativas en los COG.
* Formato de datos: CSV es el formato más rápido para ingerir, JSON normalmente tardará x2 o x3 para el mismo volumen de datos
* Ancho de tabla: asegúrese de que solo ingesta los datos que realmente necesita, ya que cuanto más amplia sea la tabla, más columnas se codificarán e indexarán, por lo tanto, menor será el rendimiento.
    Puede controlar qué campos se ingiren proporcionando una asignación de ingesta.
* Ubicación de los datos de origen: evitar las lecturas entre regiones acelera la ingestión
* Carga en el clúster: cuando el clúster experimenta una alta carga de consultas, las ingestas tardarán más en completarse, lo que reduce el rendimiento
* Patrón de ingestión: la ingestión está en su punto óptimo cuando el `KustoQueuedIngestClient`clúster se sirve con lotes de hasta 1 GB (cuidado al usar)

## <a name="optimizing-for-cogs"></a>Optimización para el COGS
Al usar las bibliotecas de cliente de Kusto para ingerir datos en Kusto sigue siendo la opción más barata y robusta, instamos a nuestros clientes a revisar sus tácticas de ingesta y tener en cuenta los recientes cambios (otoño de 2017) en los precios de Azure Storage, que encarecer significativamente las transacciones de blobs (x100).
<BR>
Es importante comprender que cuantos más fragmentos de datos (archivos/blobs/streams) envíe a Kusto, mayor será su factura mensual.
Si sigue las siguientes recomendaciones, podrá controlar mejor los costos de ingesta de Kusto:
* **Prefiere la ingesta de fragmentos de datos más grandes (hasta 1 GB de datos sin comprimir)**<br>
    Muchos equipos intentan lograr una latencia baja ingiriendo decenas de millones de pequeños fragmentos de datos, lo cual es extremadamente ineficiente y muy costoso.<br>
    Cualquier procesamiento por lotes en el lado del cliente ayudaría. 
* **Asegúrese de proporcionar al cliente Kusto.Ingest un tamaño de datos sin comprimir preciso**<br>
    Si no lo hace, puede hacer que Kusto realice transacciones de almacenamiento adicionales.
* **Evite** enviar datos para `FlushImmediately` su ingesta con la `true`marca `ingest-by` / `drop-by` establecida en , o enviar pequeños fragmentos con etiquetas establecidas.<br>
    El uso de estos impide que el servicio Kusto agregue correctamente los datos durante la ingesta y provoca transacciones de almacenamiento innecesarias después de la ingesta, lo que afecta a COGS.<br>
    Además, el uso excesivo de estos podría dar lugar a una ingesta degradada o al rendimiento de las consultas en el clúster.<br>
    