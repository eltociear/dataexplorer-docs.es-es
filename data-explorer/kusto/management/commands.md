---
title: 'Administración de comandos: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la administración de comandos en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 5685833431529af22aa4d8778d121f5a4dec16d1
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744306"
---
# <a name="commands-management"></a>Gestión de comandos

## <a name="show-commands"></a>Comandos .show 

`.show``commands` comando devuelve una tabla con comandos admin que han alcanzado un estado final. Estos comandos están disponibles para consultar durante 30 días.

La tabla Comandos tiene dos columnas con detalles de consumo de recursos de cada comando completado:
* TotalCpu: El tiempo total del reloj de la CPU (modo de usuario + modo kernel) consumido por este comando.
* ResourceUtilization: objeto que contiene toda la información de utilización de recursos relacionada con ese comando (incluido TotalCpu).

El consumo de recursos que se realiza el seguimiento incluye actualizaciones de datos, así como cualquier consulta asociada con el comando Admin actual.
Actualmente, solo algunos de los comandos Admin están cubiertos por la tabla Commands (.ingest, .set, .append, .set-or-replace, .set-or-append) y, poco a poco, se agregarán más comandos en el futuro.


* Un administrador de base de datos o un monitor de [base](../management/access-control/role-based-authorization.md) de datos pueden ver cualquier comando que se haya invocado en su base de datos.
* Otros usuarios solo pueden ver los comandos invocados por ellos.

**Sintaxis**

`.show` `commands`
 
**Ejemplo**
 
|ClientActivityId |CommandType |Texto |Base de datos |StartedOn |LastUpdatedOn |Duration |State |RootActivityId |Usuario |FailureReason |Application |Principal |TotalCpu |ResourceUtilization
|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--
|KD2RunCommand;a069f9e3-6062-4a0e-aa82-75a1b5e16fb4 |ExtentsMerge   |.merge operaciones asincrónicas ...    |DB1    |2017-09-05 11:08:07.5738569    |2017-09-05 11:08:09.1051161    |00:00:01.5312592   |Completed  |b965d809-3f3e-4f44-bd2b-5e1f49ac46c5   |Id. de la aplicación aAD 5ba8cec2-9a70-e92c98cad651  |   |Kusto.Azure.DM.Svc |aadapp-5ba8cec2-9a70-e92c98cad651  |00:00:03.5781250   |• "ScannedExtentsStatistics": "MinDataScannedTime": nulo, "MaxDataScannedTime": nulo, "CacheStatistics": "Memoria": "Misses": 2, "Hits": 20o, "Disco": "Misses": 2, "Hits": 0, "MemoryPeak": 159620640, "TotalCpu": "00:00:03.5781250" 
|Ke. RunCommand;710e08ca-2cd3-4d2d-b7bd-2738d335aa50 |DataIngestPull |.ingest en MyTableName ...   |TestDB |2017-09-04 16:00:37.0915452    |2017-09-04 16:04:37.2834555    |00:04:00.1919103   |Con error |a8986e9e-943f-81b0270d6fae4    |cooper@fabrikam.com    |Se ha eliminado la conexión del socket.   |Kusto.Explorer |aduser...    |00:00:00   |"ScannedExtentsStatistics": "MinDataScannedTime": nulo, "MaxDataScannedTime": nulo, "CacheStatistics": "Memoria": "Misses": 0, Hits": 0 ,, "Disco": "Misses": 0, "Hits": 0, "Hits": 0 , "MemoryPeak": 0, "TotalCpu": "00:00:00"" 
|KD2RunCommand;97db47e6-93e2-4306-8b7d-670f2c3307ff |ExtentsRebuild |.merge operaciones asincrónicas ...    |DB2    |2017-09-18 13:29:38.5945531    |2017-09-18 13:29:39.9451163    |00:00:01.3505632   |Completed  |d5ebb755-d5df-4e94-b240-9accdf06c2d1   |Id. de la aplicación aAD 5ba8cec2-9a70-e92c98cad651  |   |Kusto.Azure.DM.Svc |aadapp-5ba8cec2-9a70-e92c98cad651  |00:00:00.8906250   |• "ScannedExtentsStatistics": "MinDataScannedTime": nulo, "MaxDataScannedTime": nulo , "CacheStatistics": "Memoria": "Misses": 0, "Hits": 1 ", "Disk": "Misses": 0, "Hits": 0 á, "MemoryPeak": 88828560, "TotalCpu": "00:00:00.8906250" 

**Ejemplo: extraer datos específicos de la columna ResourceUtilization**

El acceso a una de las propiedades de la columna ResourceUtilization se realiza llamando a ResourcesUtilization.xxx (donde xxx es el nombre de propiedad).
Tenga en cuenta que ResourceUtilization es una columna dinámica y, por lo tanto, para trabajar con sus valores, primero se debe convertir en un tipo de datos específico (mediante una función de conversión como: tolong, toint, totimespan, ...).  

Por ejemplo:

```kusto
.show commands
| where CommandType == "TableAppend"
| where StartedOn > ago(1h)
| extend MemoryPeak = tolong(ResourcesUtilization.MemoryPeak)
| top 3 by MemoryPeak desc
| project StartedOn, MemoryPeak, TotalCpu, Text
```

|StartedOn |MemoryPeak |TotalCpu |Texto
|--|--|--|--
| 2017-09-28 12:11:27.8155381   | 800396032 | 00:00:04.5312500  | .append Server_Boots \| <let bootStartsSourceTable - SessionStarts; ...
| 2017-09-28 11:21:26.7304547   | 750063056 | 00:00:03.8218750  | .set-or-append WebUsage \| <database('CuratedDB'). WebUsage_v2 | Resumir... | Proyecto...
| 2017-09-28 12:16:17.4762522   | 676289120 | 00:00:00.0625000  | .set-or-append AtlasClusterEventStats with(..) <\| Atlas_Temp(datetime(2017-09-28 12:13:28.7621737),datetime(2017-09-28 12:14:28.8168492))

## <a name="investigating-performance-issues"></a>Investigación de problemas de rendimiento

`.show``commands` se pueden utilizar para investigar los problemas de rendimiento, ya que permiten ver los recursos consumidos por cada comando de control.
Los siguientes ejemplos son consultas simples y útiles para este tipo de investigaciones.

### <a name="querying-the-totalcpu-column"></a>Consulta de la columna TotalCpu

Las 10 consultas principales que consumen CPU en el último día:

```kusto
.show commands
| where StartedOn > ago(1d)
| top 10 by TotalCpu
| project StartedOn, CommandType, ClientActivityId, TotalCpu 
```

Todas las consultas de las últimas 10 horas que su TotalCpu ha cruzado los 3 minutos:

```kusto
.show commands
| where StartedOn > ago(10h) and TotalCpu > 3m
| project StartedOn, CommandType, ClientActivityId, TotalCpu 
| order by TotalCpu 
```

Todas las consultas de las últimas 24 horas en las que su TotalCpu estuvo por encima de 5 minutos, agrupadas por Usuario y Principal:

```kusto
.show commands  
| where StartedOn > ago(24h)
| summarize TotalCount=count(), CountAboveThreshold=countif(TotalCpu > 2m), AverageCpu = avg(TotalCpu), MaxTotalCpu=max(TotalCpu), (50th_Percentile_TotalCpu, 95th_Percentile_TotalCpu)=percentiles(TotalCpu, 50, 95) by User, Principal
| extend PercentageAboveThreshold = strcat(substring(CountAboveThreshold * 100 / TotalCount, 0, 5), "%")
| order by CountAboveThreshold desc
| project User, Principal, CountAboveThreshold, TotalCount, PercentageAboveThreshold, MaxTotalCpu, AverageCpu, 50th_Percentile_TotalCpu, 95th_Percentile_TotalCpu
```

Timechart: CPU promedio vs percentil 95 vs CPU máxima:

```kusto
.show commands 
| where StartedOn > ago(1d) 
| summarize MaxCpu_Minutes=max(TotalCpu)/1m, 95th_Percentile_TotalCpu_Minutes=percentile(TotalCpu, 95)/1m, AverageCpu_Minutes=avg(TotalCpu)/1m by bin(StartedOn, 1m)
| render timechart
```

## <a name="querying-the-memorypeak"></a>Consultar el MemoryPeak

Las 10 consultas principales del último día con los valores MemoryPeak más altos:

```kusto
.show commands
| where StartedOn > ago(1d)
| extend MemoryPeak = tolong(ResourcesUtilization.MemoryPeak)
| project StartedOn, CommandType, ClientActivityId, TotalCpu, MemoryPeak
| top 10 by MemoryPeak  
```

Timechart of Average MemoryPeak vs 95th Percentile vs Max MemoryPeak:

```kusto
.show commands 
| where StartedOn > ago(1d)
| project MemoryPeak = tolong(ResourcesUtilization.MemoryPeak), StartedOn 
| summarize Max_MemoryPeak=max(MemoryPeak), 95th_Percentile_MemoryPeak=percentile(MemoryPeak, 95), Average_MemoryPeak=avg(MemoryPeak) by bin(StartedOn, 1m)
| render timechart
```
