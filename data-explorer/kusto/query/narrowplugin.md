---
title: complemento estrecho - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe el complemento estrecho en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 75b211f32c15eefc60ca40b0408345be4a656652
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81512247"
---
# <a name="narrow-plugin"></a>plugin estrecho

```kusto
T | evaluate narrow()
```

El `narrow` complemento "despivota" una tabla ancha en una tabla con solo tres `string`columnas: número de fila, tipo de columna y valor de columna (como ).

El `narrow` plugin está diseñado principalmente para fines de visualización, ya que permite que las tablas anchas se muestren cómodamente sin necesidad de desplazamiento horizontal.

**Sintaxis**

`T | evaluate narrow()`

**Ejemplos**

En el ejemplo siguiente se muestra una manera `.show diagnostics` fácil de leer la salida del comando de control Kusto.

```kusto
.show diagnostics
 | evaluate narrow()
```

Los resultados `.show diagnostics` de sí mismo es una tabla con una sola fila y 33 columnas. Al usar `narrow` el plugin "rotamos" la salida a algo como esto:

Row  | Columna                              | Valor
-----|-------------------------------------|-----------------------------
0    | IsHealthy                           | True
0    | IsRebalanceRequerido                 | False
0    | IsScaleOutRequired                  | False
0    | MachinesTotal                       | 2
0    | MachinesOffline                     | 0
0    | NodeLastRestartedOn                 | 2017-03-14 10:59:18.9263023
0    | AdminLastElectedOn                  | 2017-03-14 10:58:41.6741934
0    | ClusterWarmDataCapacityFactor       | 0.130552847673333
0    | ExtentsTotal                        | 136
0    | DiskColdAllocationPercentage        | 5
0    | InstancesTargetBasedOnDataCapacity  | 2
0    | TotalOriginalDataSize               | 5167628070
0    | TotalExtentSize                     | 1779165230
0    | IngestionsLoadFactor                | 0
0    | IngestionesInProgress                | 0
0    | IngestionsSuccessRate               | 100
0    | MergesInProgress                    | 0
0    | BuildVersion                        | 1.0.6281.19882
0    | BuildTime                           | 2017-03-13 11:02:44.0000000
0    | ClusterDataCapacityFactor           | 0.130552847673333
0    | IsDataWarmingRequired               | False
0    | RebalanceLastRunon                  | 2017-03-21 09:14:53.8523455
0    | DataWarmingLastRunon                | 2017-03-21 09:19:54.1438800
0    | MergesSuccessRate                   | 100
0    | NotHealthyReason                    | [nulo]
0    | IsAttentionRequired                 | False
0    | AtenciónRequeridoCausa             | [nulo]
0    | ProductVersion                      | KustoRelease_2017.03.13.2
0    | FailedIngestOperations              | 0
0    | FailedMergeOperations               | 0
0    | MaxExtentsInSingleTable             | 64
0    | TableWithMaxExtents                 | KustoMonitoringPersistentDatabase.KustoMonitoringTable
0    | WarmExtentSize                      | 1779165230