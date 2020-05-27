---
title: 'Administración de comandos: Azure Explorador de datos'
description: En este artículo se describe la administración de comandos en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e6a31e2c79ae658cceecfdad0ce307e2522bb55b
ms.sourcegitcommit: 283cce0e7635a2d8ca77543f297a3345a5201395
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2020
ms.locfileid: "84011421"
---
# <a name="commands-management"></a>Administración de comandos

## <a name="show-commands"></a>. Show (comandos) 

`.show commands`Devuelve una tabla con comandos de administrador que han alcanzado un estado final. Estos comandos están disponibles para realizar consultas durante 30 días.

La tabla comandos tiene dos columnas con los detalles de consumo de recursos de cada comando completado.

* TotalCpu: el tiempo de reloj total de la CPU (modo de usuario + modo kernel) utilizado por este comando.
* ResourceUtilization: contiene toda la información de uso de recursos relacionada con ese comando, incluido TotalCpu.

El consumo de recursos del que se realiza el seguimiento incluye las actualizaciones de datos y cualquier consulta asociada al comando de administrador actual.
Actualmente, solo algunos de los comandos de administración están incluidos en la tabla de comandos ( `.ingest` , `.set` , `.append` , `.set-or-replace` , `.set-or-append` ). Gradualmente, se agregarán más comandos a la tabla Commands.

* Un [Administrador de base](../management/access-control/role-based-authorization.md) de datos o un monitor de base de datos puede ver cualquier comando que se invocó en su base de datos.
* Otros usuarios solo pueden ver los comandos que invocaron.

**Sintaxis**

`.show` `commands`
 
**Ejemplo**
 
|ClientActivityId |CommandType |Texto |Base de datos |Inicio de |LastUpdatedOn |Duration |State |RootActivityId |Usuario |FailureReason |Aplicación |Principal |TotalCpu |ResourceUtilization
|--|--|--|--|--|--|--|--|--|--|--|--|--|--|--
|KD2RunCommand;a069f9e3-6062-4a0e-aa82-75a1b5e16fb4 |ExtentsMerge   |. combinar operaciones asincrónicas...    |DB1    |2017-09-05 11:08:07.5738569    |2017-09-05 11:08:09.1051161    |00:00:01.5312592   |Completado  |b965d809-3f3e-4f44-bd2b-5e1f49ac46c5   |ID. de aplicación de AAD = 5ba8cec2-9a70-e92c98cad651  |   |Kusto. Azure. DM. SVC |aadapp = 5ba8cec2-9a70-e92c98cad651  |00:00:03.5781250   |{"ScannedExtentsStatistics": {"MinDataScannedTime": null, "MaxDataScannedTime": null}, "CacheStatistics": {Memory ": {" Errors ": 2," hits ": 20}," disco ": {" errores ": 2," aciertos ": 0}}," MemoryPeak ": 159620640," TotalCpu ":" 00:00:03.5781250 "} 
|KE. RunCommand 710e08ca-2cd3-4d2d-b7bd-2738d335aa50    |DataIngestPull |. ingerir en MyTableName...   |TestDB |2017-09-04 16:00:37.0915452    |2017-09-04 16:04:37.2834555    |00:04:00.1919103   |Failed |a8986e9e-943f-81b0270d6fae4    |cooper@fabrikam.com    |La conexión de socket se ha eliminado.   |Kusto.Explorer |aaduser =...    |00:00:00   |{"ScannedExtentsStatistics": {"MinDataScannedTime": null, "MaxDataScannedTime": null}, "CacheStatistics": {"Memory": {"Errors": 0, aciertos ": 0}," disco ": {" errores ": 0," aciertos ": 0}}," MemoryPeak ": 0," TotalCpu ":" 00:00:00 "} 
|KD2RunCommand;97db47e6-93e2-4306-8b7d-670f2c3307ff |ExtentsRebuild |. combinar operaciones asincrónicas...    |DB2    |2017-09-18 13:29:38.5945531    |2017-09-18 13:29:39.9451163    |00:00:01.3505632   |Completado  |d5ebb755-d5df-4e94-b240-9accdf06c2d1   |ID. de aplicación de AAD = 5ba8cec2-9a70-e92c98cad651  |   |Kusto. Azure. DM. SVC |aadapp = 5ba8cec2-9a70-e92c98cad651  |00:00:00.8906250   |{"ScannedExtentsStatistics": {"MinDataScannedTime": null, "MaxDataScannedTime": null}, "CacheStatistics": {Memory ": {" Errors ": 0," hits ": 1}," disco ": {" errores ": 0," aciertos ": 0}}," MemoryPeak ": 88828560," TotalCpu ":" 00:00:00.8906250 "} 

**Ejemplo: extraer datos específicos de la columna ResourceUtilization**

El acceso a una de las propiedades dentro de la columna ResourceUtilization se realiza mediante una llamada a ResourcesUtilization.xxx (donde xxx es el nombre de la propiedad).
> [!NOTE] 
> `ResourceUtilization`es una columna dinámica. Para trabajar con sus valores, primero debe convertirlos en un tipo de datos específico. Use una función de conversión como `tolong` , `toint` , `totimespan` .  

Por ejemplo,

```kusto
.show commands
| where CommandType == "TableAppend"
| where StartedOn > ago(1h)
| extend MemoryPeak = tolong(ResourcesUtilization.MemoryPeak)
| top 3 by MemoryPeak desc
| project StartedOn, MemoryPeak, TotalCpu, Text
```

|Inicio de |MemoryPeak |TotalCpu |Texto
|--|--|--|--
| 2017-09-28 12:11:27.8155381   | 800396032 | 00:00:04.5312500 |. Append Server_Boots <\| permitir bootStartsSourceTable = SessionStarts;...
| 2017-09-28 11:21:26.7304547   | 750063056 | 00:00:03.8218750 |. establece o anexa webusage <base de \| datos (' CuratedDB '). WebUsage_v2 | resumir... | proyecto...
| 2017-09-28 12:16:17.4762522   | 676289120 | 00:00:00.0625000 |. Set-or-Append AtlasClusterEventStats with (...) <\| Atlas_Temp (DateTime (2017-09-28 12:13:28.7621737), DateTime (2017-09-28 12:14:28.8168492))

## <a name="investigating-performance-issues"></a>Investigar problemas de rendimiento

`.show``commands`se puede utilizar para investigar los problemas de rendimiento, ya que muestran los recursos consumidos por cada comando de control.

Los ejemplos siguientes son consultas sencillas y útiles para estas investigaciones.

### <a name="query-the-totalcpu-column"></a>Consultar la columna TotalCpu

Las 10 primeras consultas que consumen CPU en el último día.

```kusto
.show commands
| where StartedOn > ago(1d)
| top 10 by TotalCpu
| project StartedOn, CommandType, ClientActivityId, TotalCpu 
```

Todas las consultas de las últimas 10 horas cuya TotalCpu ha pasado 3 minutos.

```kusto
.show commands
| where StartedOn > ago(10h) and TotalCpu > 3m
| project StartedOn, CommandType, ClientActivityId, TotalCpu 
| order by TotalCpu 
```

Todas las consultas de las últimas 24 horas cuya TotalCpu ha pasado 5 minutos, agrupadas por usuario y entidad de seguridad.

```kusto
.show commands  
| where StartedOn > ago(24h)
| summarize TotalCount=count(), CountAboveThreshold=countif(TotalCpu > 2m), AverageCpu = avg(TotalCpu), MaxTotalCpu=max(TotalCpu), (50th_Percentile_TotalCpu, 95th_Percentile_TotalCpu)=percentiles(TotalCpu, 50, 95) by User, Principal
| extend PercentageAboveThreshold = strcat(substring(CountAboveThreshold * 100 / TotalCount, 0, 5), "%")
| order by CountAboveThreshold desc
| project User, Principal, CountAboveThreshold, TotalCount, PercentageAboveThreshold, MaxTotalCpu, AverageCpu, 50th_Percentile_TotalCpu, 95th_Percentile_TotalCpu
```

Gráfico: promedio de CPU frente a 95 percentil frente a Max CPU.

```kusto
.show commands 
| where StartedOn > ago(1d) 
| summarize MaxCpu_Minutes=max(TotalCpu)/1m, 95th_Percentile_TotalCpu_Minutes=percentile(TotalCpu, 95)/1m, AverageCpu_Minutes=avg(TotalCpu)/1m by bin(StartedOn, 1m)
| render timechart
```

## <a name="query-the-memorypeak"></a>Consulta de MemoryPeak

Las 10 primeras consultas del último día con los valores de MemoryPeak más altos.

```kusto
.show commands
| where StartedOn > ago(1d)
| extend MemoryPeak = tolong(ResourcesUtilization.MemoryPeak)
| project StartedOn, CommandType, ClientActivityId, TotalCpu, MemoryPeak
| top 10 by MemoryPeak  
```

Gráfico media MemoryPeak vs 95 percentil vs Max MemoryPeak.

```kusto
.show commands 
| where StartedOn > ago(1d)
| project MemoryPeak = tolong(ResourcesUtilization.MemoryPeak), StartedOn 
| summarize Max_MemoryPeak=max(MemoryPeak), 95th_Percentile_MemoryPeak=percentile(MemoryPeak, 95), Average_MemoryPeak=avg(MemoryPeak) by bin(StartedOn, 1m)
| render timechart
```
