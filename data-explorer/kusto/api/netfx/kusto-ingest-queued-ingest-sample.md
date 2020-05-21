---
title: 'Ingesta de datos con la biblioteca Kusto. ingesta: Azure Explorador de datos'
description: En este artículo se describe la ingesta de datos con la biblioteca Kusto. ingesta de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/05/2020
ms.openlocfilehash: 0e6564e6c27c62621678ae350514bf1df39c73ae
ms.sourcegitcommit: ee90472a4f9d751d4049744d30e5082029c1b8fa
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/21/2020
ms.locfileid: "83722089"
---
# <a name="data-ingestion-with-the-kustoingest-library"></a>Ingesta de datos con la biblioteca Kusto. ingesta

En este artículo se presenta código de ejemplo que usa la biblioteca de cliente de Kusto. ingesta para la ingesta de datos. El código detalla el modo recomendado de ingesta para canalizaciones de nivel de producción, lo que se conoce como ingesta en cola. En la biblioteca Kusto. ingesta, la entidad correspondiente es la interfaz [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) . El código de cliente interactúa con el servicio Azure Explorador de datos mediante la publicación de notificaciones de ingesta en una cola de Azure. La referencia a la cola se obtiene de la entidad Administración de datos responsable de la ingesta. 

> [!NOTE]
> La interacción con el servicio Administración de datos se debe autenticar con Azure Active Directory (Azure AD).

En el ejemplo se usa Azure AD autenticación de usuario y se ejecuta bajo la identidad del usuario interactivo.

## <a name="dependencies"></a>Dependencias

Este código de ejemplo requiere los siguientes paquetes NuGet:
* Microsoft. Kusto. ingesta
* Microsoft.IdentityModel.Clients.ActiveDirectory
* WindowsAzure.Storage
* Newtonsoft.Json

## <a name="namespaces-used"></a>Espacios de nombres usados

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

El código hace lo siguiente.
1. Crea una tabla en el `KustoLab` clúster compartido de explorador de datos de Azure en la `KustoIngestClientDemo` base de datos.
2. Aprovisiona un [objeto de asignación de columnas JSON](../../management/create-ingestion-mapping-command.md) en esa tabla
3. Crea una instancia de [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient) para el `Ingest-KustoLab` servicio administración de datos
4. Configura [KustoQueuedIngestionProperties](kusto-ingest-client-reference.md#class-kustoqueuedingestionproperties) con las opciones de ingesta adecuadas
5. Crea un MemoryStream rellenado con algunos datos generados que se van a ingerir
6. Ingeri los datos mediante el `KustoQueuedIngestClient.IngestFromStream` método
7. Sondea los errores de [ingesta](kusto-ingest-client-status.md#tracking-ingestion-status-kustoqueuedingestclient)

```csharp
static void Main(string[] args)
{
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
        var columnMappings = new List<ColumnMapping>();
            columnMappings.Add(new ColumnMapping()
            {
                ColumnName = "Column1",
                Properties = new Dictionary<string, string>() {
                    { Data.Common.MappingConsts.Path, "$.Id" },
            } });
            columnMappings.Add(new ColumnMapping()
            {
                ColumnName = "Column2",
                Properties = new Dictionary<string, string>() {
                    { Data.Common.MappingConsts.Path, "$.Timestamp" },
            }
            });
            columnMappings.Add(new ColumnMapping()
            {
                ColumnName = "Column3",
                Properties = new Dictionary<string, string>() {
                    { Data.Common.MappingConsts.Path, "$.Message" },
            }
            });
            var secondCommand = CslCommandGenerator.GenerateTableMappingCreateCommand(
                Data.Ingestion.IngestionMappingKind.Json, table, mappingName, columnMappings);
        kustoAdminClient.ExecuteControlCommand(databaseName: db, command: secondCommand);
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
        ingestProps.IngestionMapping = new IngestionMapping()
        { 
            IngestionMappingReference = mappingName
        };
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
        var errors = ingestClient.GetAndDiscardTopIngestionFailuresAsync().GetAwaiter().GetResult();
        var successes = ingestClient.GetAndDiscardTopIngestionSuccessesAsync().GetAwaiter().GetResult();

        errors.ForEach((f) => { Console.WriteLine($"Ingestion error: {f.Info.Details}"); });
        successes.ForEach((s) => { Console.WriteLine($"Ingested: {s.Info.IngestionSourcePath}"); });
    }
}
```
