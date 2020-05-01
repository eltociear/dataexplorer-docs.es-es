---
title: 'Kusto IngestionBatching Policy optimiza el procesamiento por lotes: Azure Explorador de datos'
description: En este artículo se describe la Directiva IngestionBatching en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 8f079fd7deb6e2b1ef81565aaf591ef4a5d0f020
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617755"
---
# <a name="ingestionbatching-policy"></a>Directiva IngestionBatching

## <a name="overview"></a>Información general

Durante el proceso de ingesta, Kusto intenta optimizar el rendimiento mediante el procesamiento por lotes de fragmentos de datos de entrada pequeños junto a medida que esperan la ingesta.
Este tipo de procesamiento por lotes reduce los recursos consumidos por el proceso de ingesta, y no requiere recursos posteriores a la ingesta para optimizar las particiones de datos pequeñas que produce la ingesta sin lotes.

Sin embargo, hay un inconveniente para realizar el procesamiento por lotes antes de la ingesta, que es la introducción de un retraso forzado, de modo que el tiempo de extremo a extremo que solicita la ingesta de datos hasta que esté listo para la consulta es mayor.

Para permitir el control de este equilibrio, puede usar la `IngestionBatching` Directiva.
Esta directiva solo se aplica a la ingesta en cola y proporciona el retraso máximo forzado para permitir al procesar por lotes pequeños BLOBs.

## <a name="details"></a>Detalles

Como se explicó anteriormente, hay un tamaño óptimo de los datos que se van a ingerir en masa.
Actualmente, ese tamaño es aproximadamente 1 GB de datos sin comprimir. La ingesta realizada en blobs que contengan muchos menos datos que el tamaño óptimo no es óptimo y, por lo tanto, en la ingesta en cola Kusto, se procesarán por lotes estos blobs pequeños juntos. El procesamiento por lotes se realiza hasta que la primera condición sea verdadera:

1. El tamaño total de los datos por lotes alcanza el tamaño óptimo, o bien
2. Se alcanza el tiempo de retraso máximo, el tamaño total o el número de `IngestionBatching` blobs permitidos por la Directiva.

La `IngestionBatching` Directiva se puede establecer en las bases de datos o en las tablas. De forma predeterminada, si no se define la Directiva, Kusto usará un valor predeterminado de **5 minutos** como el tiempo máximo de retraso, **1000** elementos, el tamaño total de **1g** para el procesamiento por lotes.

> [!WARNING]
> Se recomienda que los clientes que deseen establecer esta Directiva se pongan en contacto primero con el equipo de operaciones de Kusto. El impacto de la configuración de esta directiva en un valor muy pequeño es un aumento en el costo de ventas del clúster y un rendimiento reducido. Además, en el límite, reducir este valor podría producir realmente una latencia de ingesta de **un** extremo a otro eficaz, debido a la sobrecarga que supone la administración de varios procesos de ingesta en paralelo.

## <a name="additional-resources"></a>Recursos adicionales

* [Referencia de comandos de directiva de IngestionBatching](../management/batching-policy.md)
* [Prácticas recomendadas de ingesta: optimización del rendimiento](../api/netfx/kusto-ingest-best-practices.md#optimizing-for-throughput)