---
title: 'Introducción del centro de eventos: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la introducción del centro de eventos en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: 6b2f3b2d75bd964401ae37093405e692cfd64feb
ms.sourcegitcommit: f6cf88be736aa1e23ca046304a02dee204546b6e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2020
ms.locfileid: "82861789"
---
# <a name="ingest-from-event-hub"></a>Ingesta desde centro de eventos

[Azure Event hubs](https://docs.microsoft.com/azure/event-hubs/event-hubs-about) es una plataforma de streaming de macrodatos y un servicio de ingesta de eventos. Azure Explorador de datos ofrece ingesta continua del Event Hubs administrado por el cliente. 

## <a name="data-format"></a>Formato de datos

* Los datos se leen desde el centro de eventos en forma de objetos [EventData](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata?view=azure-dotnet) .
* La carga del evento puede contener uno o varios registros que se van a ingerir, en uno de los [formatos admitidos por Azure explorador de datos](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats).
* Los datos se pueden comprimir `GZip` mediante el algoritmo de compresión. Se debe especificar como `Compression` [propiedad de ingesta](#ingestion-properties).

> [!Note]
> * La compresión de datos no se admite para los formatos comprimidos (Avro, parquet, ORC).
> * La codificación personalizada y [las propiedades del sistema](#event-system-properties-mapping) incrustado no se admiten en datos comprimidos.

## <a name="ingestion-properties"></a>Propiedades de la ingesta

Propiedades de ingesta indica el proceso de ingesta. Dónde enrutar los datos y cómo procesarlos. Puede especificar [las propiedades de ingesta](https://docs.microsoft.com/azure/data-explorer/ingestion-properties) de la ingesta de eventos mediante [EventData. Properties](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata.properties?view=azure-dotnet#Microsoft_ServiceBus_Messaging_EventData_Properties). Puede establecer las siguientes propiedades:

|Propiedad. |Descripción|
|---|---|
| Tabla | Nombre (distingue mayúsculas de minúsculas) de la tabla de destino existente. Invalida el valor de `Table` establecido en la hoja `Data Connection`. |
| Formato | Formato de datos. Invalida el valor de `Data format` establecido en la hoja `Data Connection`. |
| IngestionMappingReference | Nombre de la [asignación de ingesta](../create-ingestion-mapping-command.md) existente que se va a usar. Invalida el valor de `Column mapping` establecido en la hoja `Data Connection`.|
| Compresión | Compresión de datos `None` , (valor predeterminado `GZip` ) o compresión.|
| Encoding |  Codificación de datos, el valor predeterminado es UTF8. Puede ser cualquiera de las [codificaciones admitidas por .net](https://docs.microsoft.com/dotnet/api/system.text.encoding?view=netframework-4.8#remarks). |
| Etiquetas (vista previa) | Una lista de [etiquetas](../extents-overview.md#extent-tagging) para asociar a los datos ingeridos, con el formato de una cadena de matriz JSON. Tenga en cuenta las [implicaciones de rendimiento](../extents-overview.md#performance-notes-1) del uso de etiquetas. |

<!--| Database | Name of the existing target database.|-->
<!--| Tags | String representing [tags](https://docs.microsoft.com/azure/kusto/management/extents-overview#extent-tagging) that will be attached to resulting extent. |-->

## <a name="events-routing"></a>Enrutamiento de eventos

Al configurar una conexión del centro de eventos a Azure Explorador de datos clúster, se especifican las propiedades de la tabla de destino (nombre de tabla, formato de datos, compresión y asignación). Este es el enrutamiento predeterminado para los datos, también denominados `static routing`.
También puede especificar las propiedades de la tabla de destino para cada evento, mediante las propiedades del evento. La conexión enrutará dinámicamente los datos tal y como se especifica en [EventData. Properties](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata.properties?view=azure-dotnet#Microsoft_ServiceBus_Messaging_EventData_Properties), invalidando las propiedades estáticas de este evento.

En el ejemplo siguiente, establezca detalles del centro de eventos y envíe datos de métricas meteorológicas a la tabla `WeatherMetrics`.
Los datos están `json` en formato. `mapping1`está predefinido en la tabla `WeatherMetrics`:

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
eventData.Properties.Add("Tags", "['mydatatag']");

// Send events
var eventHubClient = EventHubClient.CreateFromConnectionString(eventHubNamespaceConnectionString, eventHubName);
eventHubClient.Send(eventData);
eventHubClient.Close();
```

## <a name="event-system-properties-mapping"></a>Asignación de propiedades del sistema de eventos

Las propiedades del sistema son una colección que se utiliza para almacenar propiedades establecidas por el servicio Event Hubs, en el momento en que se pone en cola el evento. La conexión del centro de eventos de Azure Explorador de datos incrustará las propiedades seleccionadas en el desembarque de datos de la tabla.

> [!Note]
> * Las propiedades del sistema se admiten para los eventos de registro único.
> * No se admiten las propiedades del sistema en los datos comprimidos.
> * Para `csv` la asignación, las propiedades se agregan al principio del registro en el orden que se muestra en la tabla siguiente. Para `json` la asignación, las propiedades se agregan según los nombres de propiedad de la tabla siguiente.

### <a name="event-hub-expose-the-following-system-properties"></a>Event Hub expone las siguientes propiedades del sistema

|Propiedad |Tipo de datos |Descripción|
|---|---|---|
| x-opt-enqueued-time |datetime | Hora UTC en la que se puso en cola el evento. |
| x-opt-sequence-number |long | Número de secuencia lógica del evento en el flujo de partición del centro de eventos.
| x-opt-offset |string | Desplazamiento del evento con respecto a la secuencia de partición del centro de eventos. El identificador de desplazamiento es único dentro de una partición de la secuencia del centro de eventos. |
| x-opt-Publisher |string | El nombre del publicador si el mensaje se envió a un extremo del publicador. |
| x-opt-partition-key |string |La clave de partición de la partición correspondiente que almacenó el evento. |

Si seleccionó **propiedades del sistema de eventos** en la sección **origen de datos** de la tabla, debe incluir las propiedades en el esquema y la asignación de la tabla.

**Ejemplo de esquema de tabla**

Si los datos incluyen tres columnas (`Timespan`, `Metric`y `Value`) y las propiedades que se incluyen son `x-opt-enqueued-time` y `x-opt-offset`, cree o modifique el esquema de tabla mediante este comando:

```kusto
    .create-merge table TestTable (TimeStamp: datetime, Metric: string, Value: int, EventHubEnqueuedTime:datetime, EventHubOffset:long)
```

**Ejemplo de asignación de CSV**

Ejecute los siguientes comandos para agregar datos al principio del registro. Nota valores ordinales: las propiedades se agregan al principio del registro en el orden indicado en la tabla anterior. Esto es importante para la asignación de CSV, donde los ordinales de columna cambiarán en función de las propiedades del sistema que se asignan.

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
 
**Ejemplo de asignación de JSON**

Los datos se agregan mediante los nombres de las propiedades del sistema tal y como aparecen en la lista de **propiedades del sistema de eventos** de la hoja de **conexión de datos** . Ejecute estos comandos:

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
> Para obtener el mejor rendimiento, cree todos los recursos en la misma región que el clúster de Explorador de datos de Azure.

### <a name="create-an-event-hub"></a>Creación de un Centro de eventos

Si aún no tiene una, [cree un centro de eventos](https://docs.microsoft.com/azure/event-hubs/event-hubs-create). Puede encontrar una plantilla en la guía de creación de [un centro de eventos](https://docs.microsoft.com/azure/data-explorer/ingest-data-event-hub#create-an-event-hub) .

> [!Note]
> * El número de particiones no es modificable, por lo que debería tener en cuenta la escala a largo plazo a la hora de configurar este número.
> * El grupo de consumidores *debe* ser uniqe por consumidor. Cree un grupo de consumidores dedicado a la conexión de Explorador de datos de Azure.

### <a name="data-ingestion-connection-to-azure-data-explorer"></a>Conexión de ingesta de datos a Azure Explorador de datos

* A través de Azure Portal: [Conéctese al centro de eventos](https://docs.microsoft.com/azure/data-explorer/ingest-data-event-hub#connect-to-the-event-hub).
* Uso del SDK de .NET de administración de Azure Explorador de datos: [incorporación de una conexión de datos del centro de eventos](https://docs.microsoft.com/azure/data-explorer/data-connection-event-hub-csharp#add-an-event-hub-data-connection)
* Uso del SDK de Python de Azure Explorador de datos Management: [incorporación de una conexión de datos del centro de eventos](https://docs.microsoft.com/azure/data-explorer/data-connection-event-hub-python#add-an-event-hub-data-connection)
* Con plantilla de ARM: [Azure Resource Manager plantilla para agregar una conexión de datos del centro de eventos](https://docs.microsoft.com/azure/data-explorer/data-connection-event-hub-resource-manager#azure-resource-manager-template-for-adding-an-event-hub-data-connection)

> [!Note]
> Si **mis datos incluyen** la información de enrutamiento seleccionada, *debe* proporcionar la información de [enrutamiento](#events-routing) necesaria como parte de las propiedades de eventos.

> [!Note]
> Una vez establecida la conexión, ingesta datos a partir de los eventos que se ponen en cola después de la hora de creación.

#### <a name="generating-data"></a>Generación de datos

* Consulte la [aplicación de ejemplo](https://github.com/Azure-Samples/event-hubs-dotnet-ingest) que genera datos y los envía a un centro de eventos.

Un evento puede contener uno o más registros, hasta su límite de tamaño. En el ejemplo siguiente se envían dos eventos, cada uno de los cuales tiene 5 registros anexados:

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
