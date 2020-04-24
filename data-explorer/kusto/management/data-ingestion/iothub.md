---
title: Ingerimiento de IoT Hub - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe la ingesta de IoT Hub en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: 33b6f4f737657ee0a6c2712f3b7cadce0c9c0a8f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81521359"
---
# <a name="ingest-from-iot-hub"></a>Ingerimiento de IoT Hub

[Azure IoT Hub](https://docs.microsoft.com/azure/iot-hub/about-iot-hub) es un servicio administrado, hospedado en la nube, que actúa como un centro de mensajes central para la comunicación bidireccional entre la aplicación de IoT y los dispositivos que administra. Azure Data Explorer ofrece ingesta continua desde IoT Hubs administrados por el cliente, mediante su punto de conexión integrado compatible con [Event Hub.](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-d2c#routing-endpoints)

## <a name="data-format"></a>Formato de datos

* Los datos se leen desde el punto de conexión del centro de eventos en forma de objetos [EventData.](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata?view=azure-dotnet)
* La carga de eventos puede estar en uno de los [formatos admitidos por Azure Data Explorer](https://docs.microsoft.com/azure/data-explorer/ingestion-supported-formats).

## <a name="ingestion-properties"></a>Propiedades de la ingesta

Las propiedades de ingesta indican el proceso de ingestión. Dónde enrutar los datos y cómo procesarlos. Puede especificar [las propiedades Ingestion](https://docs.microsoft.com/azure/data-explorer/ingestion-properties) de la ingesta de eventos mediante [EventData.Properties](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata.properties?view=azure-dotnet#Microsoft_ServiceBus_Messaging_EventData_Properties). Puede establecer las siguientes propiedades:

|Propiedad |Descripción|
|---|---|
| Tabla | Nombre (distingue mayúsculas y minúsculas) de la tabla de destino existente. Reemplaza el `Table` conjunto `Data Connection` de la hoja. |
| Formato | Formato de datos. Reemplaza el `Data format` conjunto `Data Connection` de la hoja. |
| IngestionMappingReference | Nombre de la asignación de [ingesta](../create-ingestion-mapping-command.md) existente que se va a utilizar. Reemplaza el `Column mapping` conjunto `Data Connection` de la hoja.|
| Encoding |  Codificación de datos, el valor predeterminado es UTF8. Puede ser cualquiera de las [codificaciones compatibles con .NET.](https://docs.microsoft.com/dotnet/api/system.text.encoding?view=netframework-4.8#remarks) |

## <a name="events-routing"></a>Enrutamiento de eventos

Al configurar una conexión de IoT Hub en el clúster de Azure Data Explorer, especifique las propiedades de la tabla de destino (nombre de tabla, formato de datos y asignación). Este es el enrutamiento predeterminado para los `static routing`datos, también denominado .
También puede especificar propiedades de tabla de destino para cada evento, mediante las propiedades de evento. La conexión enrutará dinámicamente los datos tal como se especifica en [EventData.Properties,](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata.properties?view=azure-dotnet#Microsoft_ServiceBus_Messaging_EventData_Properties)reemplazando las propiedades estáticas para este evento.

## <a name="event-system-properties-mapping"></a>Asignación de propiedades del sistema de eventos

Las propiedades del sistema son una colección que se usa para almacenar propiedades establecidas por el servicio IoT Hubs, en el momento en que se recibe el evento. La conexión de IoT Hub de Azure Data Explorer incrustará las propiedades seleccionadas en el destino de datos de la tabla.

> [!Note]
> * Para `csv` la asignación, las propiedades se agregan al principio del registro en el orden indicado en la tabla siguiente. Para `json` la asignación, las propiedades se agregan según los nombres de propiedad de la tabla siguiente.

### <a name="iot-hub-exposes-the-following-system-properties"></a>IoT Hub expone las siguientes propiedades del sistema:

|Propiedad |Descripción|
|---|---|
| message-id | Un identificador configurable por el usuario para el mensaje utilizado para patrones de solicitud y respuesta. |
| sequence-number | Un número (exclusivo para cada cola de dispositivo) asignado por IoT Hub a cada mensaje de nube a dispositivo. |
| to | Un destino especificado en los mensajes de la nube al dispositivo. |
| absolute-expiry-time | Fecha y hora de la expiración del mensaje. |
| iothub-enqueuedtime | Fecha y hora en la que IoT Hub recibió el mensaje del dispositivo a la nube. |
| correlation-id| Cadena de propiedad en un mensaje de respuesta que normalmente contiene el identificador del mensaje de la solicitud en los patrones de solicitud y respuesta. |
| user-id| Un identificador que se utiliza para especificar el origen de los mensajes. |
| iothub-ack| Un generador de mensajes de comentarios. |
| iothub-connection-device-id| Un identificador establecido por IoT Hub en los mensajes de dispositivo a nube. Contiene el deviceId del dispositivo que envió el mensaje. |
| iothub-connection-auth-generation-id| Un identificador establecido por IoT Hub en los mensajes de dispositivo a nube. Contiene el valor connectionDeviceGenerationId (como se indica en Propiedades de identidad del dispositivo) del dispositivo que envió el mensaje. |
| iothub-connection-auth-method| Un método de autenticación establecido por IoT Hub en los mensajes de dispositivo a nube. Esta propiedad contiene información sobre el método de autenticación usado para autenticar el dispositivo que envía el mensaje. |

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

## <a name="create-iot-hub-connection"></a>Crear conexión de IoT Hub

> [!Note]
> Para obtener el mejor rendimiento, cree todos los recursos de la misma región que el clúster de Azure Data Explorer.

### <a name="create-an-iot-hub"></a>Creación de un IoT Hub

Si aún no tiene uno, [cree un concentrador de mucho.](https://docs.microsoft.com/azure/data-explorer/ingest-data-iot-hub#create-an-iot-hub)

> [!Note]
> * El `device-to-cloud partitions` recuento no se puede cambiar, por lo que debe tener en cuenta la escala a largo plazo al establecer el recuento de particiones.
> * El grupo de consumidores *debe* ser uniqe por consumidor. Cree un grupo de consumidores dedicado a la conexión Kusto. Busque el recurso en Azure `Built-in endpoints` Portal y vaya a agregar un nuevo grupo de consumidores.

### <a name="data-ingestion-connection-to-azure-data-explorer"></a>Conexión de ingesta de datos a Azure Data Explorer

* A través de Azure Portal: [conecte la tabla de Azure Data Explorer a IoT Hub.](https://docs.microsoft.com/azure/data-explorer/ingest-data-iot-hub#connect-azure-data-explorer-table-to-iot-hub)
* Uso del SDK de .NET de administración de Azure Data Explorer: agregue una conexión de datos de [IoT Hub](https://docs.microsoft.com/azure/data-explorer/data-connection-iot-hub-csharp#add-an-iot-hub-data-connection)
* Uso del SDK de Python de administración de Azure Data Explorer: agregue una conexión de [datos de IoT Hub](https://docs.microsoft.com/azure/data-explorer/data-connection-iot-hub-python#add-an-iot-hub-data-connection)
* Con plantilla ARM: [plantilla de Azure Resource Manager para agregar una conexión](https://docs.microsoft.com/azure/data-explorer/data-connection-iot-hub-resource-manager#azure-resource-manager-template-for-adding-an-iot-hub-data-connection) de datos de Iot Hub

> [!Note]
> Si **Mis datos incluyen la información** de enrutamiento seleccionada, *debe* proporcionar la información de [enrutamiento](#events-routing) necesaria como parte de las propiedades de eventos.

> [!Note]
> Una vez establecida la conexión, se ingente datos a partir de eventos en cola después de su hora de creación.

### <a name="generating-data"></a>Generación de datos

* Consulte el proyecto de [ejemplo](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/iot-hub/Quickstarts/simulated-device) que simula un dispositivo y genera datos.