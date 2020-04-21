---
title: Cómoingestión de datos con la biblioteca Kusto.Ingest - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe cómo ingestión de datos con la biblioteca Kusto.Ingest en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/05/2020
ms.openlocfilehash: 80b2b61c70269c5bd166a064fe9d0e2c59dd8197
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523637"
---
# <a name="howto-data-ingestion-with-kustoingest-library"></a>HowTo Data Ingestion with Kusto.Ingest Library
En este artículo se presenta código de ejemplo que hace uso de la biblioteca de cliente Kusto.Ingest.

## <a name="overview"></a>Información general
En el ejemplo de código siguiente se muestra la ingesta de datos en cola (que pasa a través del servicio de administración de datos de Kusto) a Kusto con el uso de la biblioteca Kusto.Ingest.

> En este artículo se trata del modo recomendado de ingestión para canalizaciones de nivel de producción, que también se conoce como **ingesta** en cola (en términos de la biblioteca Kusto.Ingest, la entidad correspondiente es la interfaz [IKustoQueuedIngestClient).](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) En este modo, el código de cliente interactúa con el servicio Kusto publicando mensajes de notificación de ingesta en una cola de Azure, referencia a la que se obtiene de Kusto Data Management (también. Ingestión). La interacción con el servicio de administración de datos debe autenticarse con **AAD**.

#### <a name="authentication"></a>Authentication
Este ejemplo de código usa la autenticación de usuario de AAD y se ejecuta bajo la identidad del usuario interactivo.

## <a name="dependencies"></a>Dependencias
Este código de ejemplo requiere los siguientes paquetes NuGet:
* Microsoft.Kusto.Ingest
* Microsoft.IdentityModel.Clients.ActiveDirectory
* WindowsAzure.Storage
* Newtonsoft.Json

## <a name="namespaces-used"></a>Espacios de nombres utilizados
```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Threading;
using Kusto.Data;
using Kusto.Data.Common;
using Kusto.Data.Net.Client;
using Kusto.Ingest;
```

## <a name="code"></a>Código
El código que se presenta a continuación realiza lo siguiente:
1. Crea una `KustoLab` tabla en el `KustoIngestClientDemo` clúster de Kusto compartido en la base de datos
2. Aprovisiona un objeto de [asignación](../../management/create-ingestion-mapping-command.md) de columnas JSON en esa tabla
3. Crea una instancia [de IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) para el `Ingest-KustoLab` servicio de administración de datos
4. Configura [KustoQueuedIngestionProperties](kusto-ingest-client-reference.md#class-kustoqueuedingestionproperties) con las opciones de ingesta adecuadas
5. Crea un MemoryStream lleno de algunos datos generados que se van a ingerir
6. Ingenia los `KustoQueuedIngestClient.IngestFromStream` datos usando el método
7. Encuestas para [cualquier error de ingesta](kusto-ingest-client-status.md#tracking-ingestion-status-kustoqueuedingestclient)

```csharp
static void Main(string[] args)
{
    var clusterName = "KustoLab";
    var db = "KustoIngestClientDemo";
    var table = "Table1";
    var mappingName = "Table1_mapping_1";
    var serviceNameAndRegion = "clusterNameAndRegion"; // For example, "mycluster.westus"
    var authority = "AAD Tenant or name"; // For example, "microsoft.com"

    // Set up table
    var kcsbEngine =
        new KustoConnectionStringBuilder($"https://{serviceNameAndRegion}.kusto.windows.net").WithAadUserPromptAuthentication(authority: $"{authority}");

    using (var kustoAdminClient = KustoClientFactory.CreateCslAdminProvider(kcsbEngine))
    {
        var columns = new List<Tuple<string, string>>()
        {
            new Tuple<string, string>("Column1", "System.Int64"),
            new Tuple<string, string>("Column2", "System.DateTime"),
            new Tuple<string, string>("Column3", "System.String"),
        };

        var command = CslCommandGenerator.GenerateTableCreateCommand(table, columns);
        kustoAdminClient.ExecuteControlCommand(databaseName: db, command: command);

        // Set up mapping
        var columnMappings = new List<JsonColumnMapping>();
        columnMappings.Add(new JsonColumnMapping()
            { ColumnName = "Column1", JsonPath = "$.Id" });
        columnMappings.Add(new JsonColumnMapping()
            { ColumnName = "Column2", JsonPath = "$.Timestamp" });
        columnMappings.Add(new JsonColumnMapping()
            { ColumnName = "Column3", JsonPath = "$.Message" });

        command = CslCommandGenerator.GenerateTableJsonMappingCreateCommand(
                                            table, mappingName, columnMappings);
        kustoAdminClient.ExecuteControlCommand(databaseName: db, command: command);
    }

    // Create Ingest Client
    var kcsbDM =
        new KustoConnectionStringBuilder($"https://ingest-{serviceNameAndRegion}.kusto.windows.net").WithAadUserPromptAuthentication(authority: $"{authority}");

    using (var ingestClient = KustoIngestFactory.CreateQueuedIngestClient(kcsbDM))
    {
        var ingestProps = new KustoQueuedIngestionProperties(db, table);
        // For the sake of getting both failure and success notifications we set this to IngestionReportLevel.FailuresAndSuccesses
        // Usually the recommended level is IngestionReportLevel.FailuresOnly
        ingestProps.ReportLevel = IngestionReportLevel.FailuresAndSuccesses;
        ingestProps.ReportMethod = IngestionReportMethod.Queue;
        ingestProps.JSONMappingReference = mappingName;
        ingestProps.Format = DataSourceFormat.json;

        // Prepare data for ingestion
        using (var memStream = new MemoryStream())
        using (var writer = new StreamWriter(memStream))
        {
            for (int counter = 1; counter <= 10; ++counter)
            {
                writer.WriteLine(
                    "{{ \"Id\":\"{0}\", \"Timestamp\":\"{1}\", \"Message\":\"{2}\" }}",
                    counter, DateTime.UtcNow.AddSeconds(100 * counter),
                    $"This is a dummy message number {counter}");
            }

            writer.Flush();
            memStream.Seek(0, SeekOrigin.Begin);

            // Post ingestion message
            ingestClient.IngestFromStream(memStream, ingestProps, leaveOpen: true);
        }

        // Wait and retrieve all notifications
        //  - Actual duration should be decided based on the effective Ingestion Batching Policy set on the table/database
        Thread.Sleep(<timespan>);
        var errors = ingestClient.GetAndDiscardTopIngestionFailures().GetAwaiter().GetResult();
        var successes = ingestClient.GetAndDiscardTopIngestionSuccesses().GetAwaiter().GetResult();

        errors.ForEach((f) => { Console.WriteLine($"Ingestion error: {f.Info.Details}"); });
        successes.ForEach((s) => { Console.WriteLine($"Ingested: {s.Info.IngestionSourcePath}"); });
    }
}
```