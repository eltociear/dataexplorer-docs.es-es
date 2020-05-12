---
title: 'Uso de las bibliotecas de cliente de .NET de PowerShell: Azure Explorador de datos'
description: En este artículo se describe el uso de las bibliotecas de cliente de .NET de PowerShell en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 05/29/2019
ms.openlocfilehash: 5c521d7e63d58dd32425e759b0cf09a22a050b40
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226047"
---
# <a name="using-the-net-client-libraries-from-powershell"></a>Uso de las bibliotecas de cliente de .NET de PowerShell

Los scripts de PowerShell pueden usar las bibliotecas de cliente de Azure Explorador de datos .NET a través de la integración integrada de PowerShell con bibliotecas .NET arbitrarias (que no son de PowerShell).

## <a name="getting-the-net-client-libraries-for-scripting-with-powershell"></a>Obtención de las bibliotecas de cliente de .NET para scripting con PowerShell

Para empezar a trabajar con las bibliotecas de cliente de Azure Explorador de datos .NET mediante PowerShell.

1. Descargue el [ `Microsoft.Azure.Kusto.Tools` paquete de NuGet](https://www.nuget.org/packages/Microsoft.Azure.Kusto.Tools/).
1. Extraiga el contenido del directorio ' Tools ' en el paquete (use una herramienta de archivo como `7-zip` ).
1. Llame `[System.Reflection.Assembly]::LoadFrom("path")` a desde PowerShell para cargar la biblioteca necesaria. 
    - El `path` parámetro del comando debe indicar la ubicación de los archivos extraídos.
1. Una vez cargados todos los ensamblados .NET dependientes:
   1. Cree una cadena de conexión de Kusto.
   1. Cree una instancia de un *proveedor de consultas* o de un *proveedor de administración*.
   1. Ejecute las consultas o los comandos, tal y como se muestra en los [ejemplos](powershell.md#examples) siguientes.

Para obtener más información, consulte las [bibliotecas de cliente de Azure explorador de datos](../netfx/about-kusto-data.md).

## <a name="examples"></a>Ejemplos

### <a name="initialization"></a>Inicialización

```powershell
#  Part 1 of 3
#  ------------
#  Packages location - This is an example of the location from where you extract the Microsoft.Azure.Kusto.Tools package.
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

#   Option A: using Azure AD User Authentication
$kcsb = New-Object Kusto.Data.KustoConnectionStringBuilder ($clusterUrl, $databaseName)
 
#   Option B: using Azure AD application Authentication
#     $applicationId = "application ID goes here"
#     $applicationKey = "application key goes here"
#     $authority = "authority goes here"
#     $kcsb = $kcsb.WithAadApplicationKeyAuthentication($applicationId, $applicationKey, $authority)
```

### <a name="example-running-an-admin-command"></a>Ejemplo: ejecutar un comando de administración

```powershell
$adminProvider = [Kusto.Data.Net.Client.KustoClientFactory]::CreateCslAdminProvider($kcsb)
$command = [Kusto.Data.Common.CslCommandGenerator]::GenerateDiagnosticsShowCommand()
Write-Host "Executing command: '$command' with connection string: '$($kcsb.ToString())'"
$reader = $adminProvider.ExecuteControlCommand($command)
$reader.Read() # this reads a single row/record. If you have multiple ones returned, you can read in a loop 
$isHealthy = $Reader.GetBoolean(0)
Write-Host "IsHealthy = $isHealthy"
```

Y el resultado es:
```
IsHealthy = True
```

### <a name="example-running-a-query"></a>Ejemplo: ejecutar una consulta

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

Y el resultado es:

|StartTime           |EndTime             |EpisodeID |EventId |Estado          |EventType         |InjuriesDirect |InjuriesIndirect |DeathsDirect |DeathsIndirect
|---------           |-------             |--------- |------- |-----          |---------         |-------------- |---------------- |------------ |--------------
|2007-12-30 16:00:00 |2007-12-30 16:05:00 |    11749 |  64588 |Georgia        |Viento de tormenta |             0 |               0 |           0 |             0
|2007-12-20 07:50:00 |2007-12-20 07:53:00 |    12554 |  68796 |MISSISSIPPI    |Viento de tormenta |             0 |               0 |           0 |             0
|2007-09-29 08:11:00 |2007-09-29 08:11:00 |    11091 |  61032 |SUR DE ATLÁNTICO |Agua spout       |             0 |               0 |           0 |             0
|2007-09-20 21:57:00 |2007-09-20 22:05:00 |    11078 |  60913 |FLORIDA        |Tornado           |             0 |               0 |           0 |             0
|2007-09-18 20:00:00 |2007-09-19 18:00:00 |    11074 |  60904 |FLORIDA        |Lluvia intensa        |             0 |               0 |           0 |             0
