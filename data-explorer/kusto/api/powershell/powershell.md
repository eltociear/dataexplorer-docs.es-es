---
title: Uso de las bibliotecas de cliente de .NET desde PowerShell - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe el uso de las bibliotecas de cliente de .NET de PowerShell en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 05/29/2019
ms.openlocfilehash: 635a23021a1a8c30347bfa27ecd65886b46a6fea
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81503220"
---
# <a name="using-the-net-client-libraries-from-powershell"></a>Uso de las bibliotecas de cliente de .NET de PowerShell

Los scripts de PowerShell pueden usar las bibliotecas de cliente de Azure Data Explorer .NET a través de la integración integrada de PowerShell con bibliotecas de .NET arbitrarias (que no son powerShell).

## <a name="getting-the-net-client-libraries-for-scripting-with-powershell"></a>Obtener las bibliotecas de cliente de .NET para secuencias de comandos con PowerShell

Para empezar a trabajar con las bibliotecas de cliente de .NET de Azure Data Explorer mediante PowerShell:

1. Descargue el [ `Microsoft.Azure.Kusto.Tools` paquete NuGet](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Tools/).
2. Extraiga el contenido del directorio 'tools' en el paquete (por ejemplo, mediante 7-zip).
3. Llame `[System.Reflection.Assembly]::LoadFrom("path")` desde powershell para cargar la biblioteca necesaria. 
    - El `"path"` parámetro del comando debe indicar la ubicación de los archivos extraídos.
4. Una vez cargados todos los ensamblados .NET dependientes, cree una cadena de conexión de Kusto, cree una instancia de un proveedor de *consultas* o un proveedor de *administración*y ejecute las consultas o comandos (como se muestra en los [ejemplos](powershell.md#examples) siguientes).

Para obtener información detallada, consulte las bibliotecas de cliente de [Azure Data Explorer.](../netfx/about-kusto-data.md)

## <a name="examples"></a>Ejemplos

### <a name="initialization"></a>Inicialización

```powershell
#  Part 1 of 3
#  ------------
#  Packages location - This is an example to the location where you extract the Microsoft.Azure.Kusto.Tools package.
#  Please make sure you load the types from a local directory and not from a remote share.
$packagesRoot = "C:\Microsoft.Azure.Kusto.Tools\Tools"

#  Part 2 of 3
#  ------------
#  Loading the Kusto.Client library and its dependencies
dir $packagesRoot\* | Unblock-File
[System.Reflection.Assembly]::LoadFrom("$packagesRoot\Kusto.Data.dll")

#  Part 3 of 3
#  ------------
#  Defining the connection to your cluster / database
$clusterUrl = "https://help.kusto.windows.net;Fed=True"
$databaseName = "Samples"

#   Option A: using AAD User Authentication
$kcsb = New-Object Kusto.Data.KustoConnectionStringBuilder ($clusterUrl, $databaseName)
 
#   Option B: using AAD application Authentication
#     $applicationId = "application ID goes here"
#     $applicationKey = "application key goes here"
#     $authority = "authority goes here"
#     $kcsb = $kcsb.WithAadApplicationKeyAuthentication($applicationId, $applicationKey, $authority)
```

### <a name="example-running-an-admin-command"></a>Ejemplo: Ejecución de un comando admin

```powershell
$adminProvider = [Kusto.Data.Net.Client.KustoClientFactory]::CreateCslAdminProvider($kcsb)
$command = [Kusto.Data.Common.CslCommandGenerator]::GenerateDiagnosticsShowCommand()
Write-Host "Executing command: '$command' with connection string: '$($kcsb.ToString())'"
$reader = $adminProvider.ExecuteControlCommand($command)
$reader.Read() # this reads a single row/record. If you have multiple ones returned, you can read in a loop 
$isHealthy = $Reader.GetBoolean(0)
Write-Host "IsHealthy = $isHealthy"
```

Y la salida es:
```
IsHealthy = True
```

### <a name="example-running-a-query"></a>Ejemplo: Ejecución de una consulta

```powershell
$queryProvider = [Kusto.Data.Net.Client.KustoClientFactory]::CreateCslQueryProvider($kcsb)
$query = "StormEvents | limit 5"
Write-Host "Executing query: '$query' with connection string: '$($kcsb.ToString())'"
#   Optional: set a client request ID and set a client request property (e.g. Server Timeout)
$crp = New-Object Kusto.Data.Common.ClientRequestProperties
$crp.ClientRequestId = "MyPowershellScript.ExecuteQuery." + [Guid]::NewGuid().ToString()
$crp.SetOption([Kusto.Data.Common.ClientRequestProperties]::OptionServerTimeout, [TimeSpan]::FromSeconds(30))

#   Execute the query
$reader = $queryProvider.ExecuteQuery($query, $crp)

# Do something with the result datatable, for example: print it formatted as a table, sorted by the 
# "StartTime" column, in descending order
$dataTable = [Kusto.Cloud.Platform.Data.ExtendedDataReader]::ToDataSet($reader).Tables[0]
$dataView = New-Object System.Data.DataView($dataTable)
$dataView | Sort StartTime -Descending | Format-Table -AutoSize
```

Y la salida es:

|StartTime           |EndTime             |EpisodeId |EventId |State          |EventType         |LesionesDirect |LesionesInasInas |DeathsDirect |MuertesIna
|---------           |-------             |--------- |------- |-----          |---------         |-------------- |---------------- |------------ |--------------
|2007-12-30 16:00:00 |2007-12-30 16:05:00 |    11749 |  64588 |Georgia        |Viento de tormenta |             0 |               0 |           0 |             0
|2007-12-20 07:50:00 |2007-12-20 07:53:00 |    12554 |  68796 |Mississippi    |Viento de tormenta |             0 |               0 |           0 |             0
|2007-09-29 08:11:00 |2007-09-29 08:11:00 |    11091 |  61032 |ATLANTIC SOUTH |Waterspout        |             0 |               0 |           0 |             0
|2007-09-20 21:57:00 |2007-09-20 22:05:00 |    11078 |  60913 |Florida        |Tornado           |             0 |               0 |           0 |             0
|2007-09-18 20:00:00 |2007-09-19 18:00:00 |    11074 |  60904 |Florida        |Lluvia fuerte        |             0 |               0 |           0 |             0