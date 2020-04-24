---
title: Directiva ingestionBatching - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe la directiva IngestionBatching en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: fa3a4cfd512e3e37ba6b6abf8f8316d6ff12fdec
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522243"
---
# <a name="ingestionbatching-policy"></a>Política de IngestionBatching

## <a name="overview"></a>Información general

Durante el proceso de ingesta, Kusto intenta optimizar el rendimiento mediante el procesamiento por lotes de pequeños fragmentos de datos de entrada a medida que esperan la ingesta.
Este tipo de procesamiento por lotes reduce los recursos consumidos por el proceso de ingesta, así como no requiere recursos posteriores a la ingesta para optimizar las particiones de datos pequeñas producidas por la ingesta no por lotes.

Sin embargo, hay una desventaja para realizar el procesamiento por lotes antes de la ingesta, que es la introducción de un retraso forzado, de modo que la hora de extremo a extremo de solicitar la ingesta de datos hasta que esté listo para la consulta es mayor.

Para permitir el control de esta compensación, se puede utilizar la `IngestionBatching` política.
Esta directiva se aplica solo a la ingesta en cola y proporciona el retraso forzado máximo para permitir el procesamiento por lotes de blobs pequeños juntos.

## <a name="details"></a>Detalles

Como se explicó anteriormente, hay un tamaño óptimo de datos que se deben ingerir a granel.
Actualmente ese tamaño es de aproximadamente 1 GB de datos sin comprimir. La ingesta que se realiza en blobs que contienen muchos menos datos que el tamaño óptimo no es óptima y, por lo tanto, en la ingesta en cola Kusto agrupará por lotes estos blobs pequeños. El procesamiento por lotes se realiza hasta que la primera condición se convierte en verdadera:

1. El tamaño total de los datos por lotes alcanza el tamaño óptimo, o
2. Se alcanza el tiempo de retardo máximo, el `IngestionBatching` tamaño total o el número de blobs permitidos por la directiva

La `IngestionBatching` directiva se puede establecer en bases de datos o tablas. De forma predeterminada, si no se define la directiva, Kusto usará un valor predeterminado de **5 minutos** como el tiempo de retardo máximo, **1000** elementos, tamaño total de **1G** para el procesamiento por lotes.

> [!WARNING]
> Se recomienda que los clientes que deseen establecer esta directiva se ponga primero en contacto con el equipo de operaciones de Kusto. El impacto de establecer esta directiva en un valor muy pequeño es un aumento en el costo de los productos vendidos del clúster y un rendimiento reducido. Además, en el límite, la reducción de este valor podría dar lugar a una **mayor** latencia de ingesta efectiva de extremo a extremo, debido a la sobrecarga de administrar varios procesos de ingesta en paralelo.

## <a name="additional-resources"></a>Recursos adicionales

* [Referencia de comandos de directiva IngestionBatching](../management/batching-policy.md)
* [Prácticas recomendadas de ingestión : optimización del rendimiento](../api/netfx/kusto-ingest-best-practices.md#optimizing-for-throughput)