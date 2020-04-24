---
title: Ingerimiento desde el centro de eventos - Explorador de azure Data Explorer Microsoft Docs
description: En este artículo se describe la ingesta desde el centro de eventos en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: ddb218e707152f529e5b547ffe06c4d3c7614811
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521512"
---
# <a name="ingest-from-event-hub"></a>Ingerimiento desde el centro de eventos

[Azure Event Hubs](https://docs.microsoft.com/azure/event-hubs/event-hubs-about) es una plataforma de streaming de big data y un servicio de ingesta de eventos. Azure Data Explorer ofrece ingesta continua desde Event Hubs administrados por el cliente. 

## <a name="data-format"></a>Formato de datos

* Los datos se leen desde el centro de eventos en forma de [EventData](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata?view=azure-dotnet) objetos.
* La carga de eventos puede contener uno o varios registros que se van a ingerir, en uno de los [formatos admitidos por Azure Data Explorer](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats).
* Los datos se `GZip` pueden comprimir mediante un algoritmo de compresión. Debe especificarse `Compression` como [propiedad de ingesta](#ingestion-properties).

> [!Note]
> * La compresión de datos no es compatible con formatos comprimidos (Avro, Parquet, ORC).
> * La codificación personalizada y [las propiedades](#event-system-properties-mapping) del sistema incrustado no se admiten en los datos comprimidos.

## <a name="ingestion-properties"></a>Propiedades de la ingesta

Las propiedades de ingesta indican el proceso de ingestión. Dónde enrutar los datos y cómo procesarlos. Puede especificar [las propiedades Ingestion](https://docs.microsoft.com/azure/data-explorer/ingestion-properties) de la ingesta de eventos mediante [EventData.Properties](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata.properties?view=azure-dotnet#Microsoft_ServiceBus_Messaging_EventData_Properties). Puede establecer las siguientes propiedades:

|Propiedad. |Descripción|
|---|---|
| Tabla | Nombre (distingue mayúsculas y minúsculas) de la tabla de destino existente. Reemplaza el `Table` conjunto `Data Connection` de la hoja. |
| Formato | Formato de datos. Reemplaza el `Data format` conjunto `Data Connection` de la hoja. |
| IngestionMappingReference | Nombre de la asignación de [ingesta](../create-ingestion-mapping-command.md) existente que se va a utilizar. Reemplaza el `Column mapping` conjunto `Data Connection` de la hoja.|
| Compresión | Compresión `None` de datos, `GZip` (predeterminado) o compresión.|
| Encoding |  Codificación de datos, el valor predeterminado es UTF8. Puede ser cualquiera de las [codificaciones compatibles con .NET.](https://docs.microsoft.com/dotnet/api/system.text.encoding?view=netframework-4.8#remarks) |

<!--| Database | Name of the existing target database.|-->
<!--| Tags | String representing [tags](https://docs.microsoft.com/azure/kusto/management/extents-overview#extent-tagging) that will be attached to resulting extent. |-->

## <a name="events-routing"></a>Enrutamiento de eventos

Al configurar una conexión de Event Hub con el clúster de Azure Data Explorer, especifique las propiedades de la tabla de destino (nombre de tabla, formato de datos, compresión y asignación). Este es el enrutamiento predeterminado para los `static routing`datos, también denominado .
También puede especificar propiedades de tabla de destino para cada evento, mediante las propiedades de evento. La conexión enrutará dinámicamente los datos tal como se especifica en [EventData.Properties,](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata.properties?view=azure-dotnet#Microsoft_ServiceBus_Messaging_EventData_Properties)reemplazando las propiedades estáticas para este evento.

En el ejemplo siguiente, establezca los detalles `WeatherMetrics`del centro de eventos y envíe los datos de las métricas meteorológicas a la tabla.
Los datos `json` están en formato. `mapping1`está predefinida en `WeatherMetrics`la tabla:

```csharp
var eventHubNamespaceConnectionString=<connection_string>;
var eventHubName=<event_hub>;

// Create the data
var metric = new Metric { Timestamp = DateTime.UtcNow, MetricName = "Temperature", Value = 32 }; 
var data = JsonConvert.SerializeObject(metric);

// Create the event and add optional "dynamic routing" properties
var eventData = new EventData(Encoding.UTF8.GetBytes(data));
eventData.Properties.Add("Table", "WeatherMetrics");
eventData.Properties.Add("Format", "json");
eventData.Properties.Add("IngestionMappingReference", "mapping1");

// Send events
var eventHubClient = EventHubClient.CreateFromConnectionString(eventHubNamespaceConnectionString, eventHubName);
eventHubClient.Send(eventData);
eventHubClient.Close();
```

## <a name="event-system-properties-mapping"></a>Asignación de propiedades del sistema de eventos

Las propiedades del sistema son una colección que se usa para almacenar las propiedades establecidas por el servicio Event Hubs, en el momento en que se pone en cola el evento. La conexión del Centro de eventos de Azure Data Explorer incrustará las propiedades seleccionadas en el destino de datos de la tabla.

> [!Note]
> * Las propiedades del sistema se admiten para eventos de registro único.
> * Las propiedades del sistema no se admiten en los datos comprimidos.
> * Para `csv` la asignación, las propiedades se agregan al principio del registro en el orden indicado en la tabla siguiente. Para `json` la asignación, las propiedades se agregan según los nombres de propiedad de la tabla siguiente.

### <a name="event-hub-expose-the-following-system-properties"></a>Event Hub expone las siguientes propiedades del sistema

|Propiedad |Tipo de datos |Descripción|
|---|---|---|
| x-opt-enqueued-time |datetime | Hora UTC cuando se puso en cola el evento. |
| x-opt-sequence-number |long | El número de secuencia lógica del evento dentro de la secuencia de particiones del centro de eventos.
| x-opt-offset |string | Desplazamiento del evento en relación con la secuencia de particiones del centro de eventos. El identificador de desplazamiento es único dentro de una partición de la secuencia de Event Hub. |
| x-opt-publisher |string | El nombre del publicador si el mensaje se envió a un punto de conexión del editor. |
| x-opt-partition-key |string |La clave de partición de la partición correspondiente que almacenó el evento. |

Si seleccionó Propiedades del **sistema** de eventos en la sección **Origen** de datos de la tabla, debe incluir las propiedades en el esquema de tabla y la asignación.

**Ejemplo de esquema de tabla**

Si los datos incluyen`Timespan` `Metric`tres `Value`columnas ( , , `x-opt-enqueued-time` `x-opt-offset`y ) y las propiedades que incluye son y , crear o modificar el esquema de tabla mediante este comando:

```kusto
    .create-merge table TestTable (TimeStamp: datetime, Metric: string, Value: int, EventHubEnqueuedTime:datetime, EventHubOffset:long)
```

**Ejemplo de mapeo CSV**

Ejecute los siguientes comandos para agregar datos al principio del registro. Nota: las propiedades se agregan al principio del registro en el orden indicado en la tabla anterior. Esto es importante para la asignación CSV donde los ordinales de columna cambiarán en función de las propiedades del sistema que se asignan.

```kusto
    .create table TestTable ingestion csv mapping "CsvMapping1"
    '['
    '   { "column" : "Timespan", "Properties":{"Ordinal":"2"}},'
    '   { "column" : "Metric", "Properties":{"Ordinal":"3"}},'
    '   { "column" : "Value", "Properties":{"Ordinal":"4"}},'
    '   { "column" : "EventHubEnqueuedTime", "Properties":{"Ordinal":"0"}},'
    '   { "column" : "EventHubOffset", "Properties":{"Ordinal":"1"}}'
    ']'
```
 
**Ejemplo de mapeo JSON**

Los datos se agregan utilizando los nombres de propiedades del sistema tal como aparecen en la hoja **Conexión** de datos Lista de propiedades del sistema de **eventos.** Ejecute estos comandos:

```kusto
    .create table TestTable ingestion json mapping "JsonMapping1"
    '['
    '    { "column" : "Timespan", "Properties":{"Path":"$.timestamp"}},'
    '    { "column" : "Metric", "Properties":{"Path":"$.metric"}},'
    '    { "column" : "Value", "Properties":{"Path":"$.metric_value"}},'
    '    { "column" : "EventHubEnqueuedTime", "Properties":{"Path":"$.x-opt-enqueued-time"}},'
    '    { "column" : "EventHubOffset", "Properties":{"Path":"$.x-opt-offset"}}'
    ']'
```

## <a name="create-event-hub-connection"></a>Creación de una conexión de Event Hubs

> [!Note]
> Para obtener el mejor rendimiento, cree todos los recursos de la misma región que el clúster de Azure Data Explorer.

### <a name="create-an-event-hub"></a>Creación de un Centro de eventos

Si aún no tiene uno, cree un centro de [eventos.](https://docs.microsoft.com/azure/event-hubs/event-hubs-create) Puede encontrar una plantilla en la guía [cómo crear un centro](https://docs.microsoft.com/azure/data-explorer/ingest-data-event-hub#create-an-event-hub) de eventos.

> [!Note]
> * El número de particiones no es modificable, por lo que debería tener en cuenta la escala a largo plazo a la hora de configurar este número.
> * El grupo de consumidores *debe* ser uniqe por consumidor. Cree un grupo de consumidores dedicado a la conexión de Azure Data Explorer.

### <a name="data-ingestion-connection-to-azure-data-explorer"></a>Conexión de ingesta de datos a Azure Data Explorer

* A través de Azure Portal: [conéctese al centro de eventos.](https://docs.microsoft.com/azure/data-explorer/ingest-data-event-hub#connect-to-the-event-hub)
* Uso del SDK de .NET de administración de Azure Data Explorer: [agregue una conexión](https://docs.microsoft.com/azure/data-explorer/data-connection-event-hub-csharp#add-an-event-hub-data-connection) de datos de Event Hub
* Uso del SDK de Python de administración de Azure Data Explorer: [agregue una conexión](https://docs.microsoft.com/azure/data-explorer/data-connection-event-hub-python#add-an-event-hub-data-connection) de datos de Event Hub
* Con plantilla ARM: [plantilla de Azure Resource Manager para agregar una conexión](https://docs.microsoft.com/azure/data-explorer/data-connection-event-hub-resource-manager#azure-resource-manager-template-for-adding-an-event-hub-data-connection) de datos de Event Hub

> [!Note]
> Si **Mis datos incluyen la información** de enrutamiento seleccionada, *debe* proporcionar la información de [enrutamiento](#events-routing) necesaria como parte de las propiedades de eventos.

> [!Note]
> Una vez establecida la conexión, se ingente datos a partir de eventos en cola después de su hora de creación.

#### <a name="generating-data"></a>Generación de datos

* Consulte la aplicación de [ejemplo](https://github.com/Azure-Samples/event-hubs-dotnet-ingest) que genera datos y los envía a un centro de eventos.

Un evento puede contener uno o varios registros, hasta su límite de tamaño. En el ejemplo siguiente enviamos dos eventos, cada uno tiene 5 registros anexados:

```csharp
var events = new List<EventData>();
var data = string.Empty;
var recordsPerEvent = 5;
var rand = new Random();
var counter = 0;

for (var i = 0; i < 10; i++)
{
    // Create the data
    var metric = new Metric { Timestamp = DateTime.UtcNow, MetricName = "Temperature", Value = rand.Next(-30, 50) }; 
    var data += JsonConvert.SerializeObject(metric) + Environment.NewLine;
    counter++;

    // Create the event
    if (counter == recordsPerEvent)
    {
        var eventData = new EventData(Encoding.UTF8.GetBytes(data));
        events.Add(eventData);

        counter = 0;
        data = string.Empty;
    }
}

// Send events
eventHubClient.SendAsync(events).Wait();
```