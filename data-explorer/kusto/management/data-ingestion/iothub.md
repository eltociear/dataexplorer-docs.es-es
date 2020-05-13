---
title: Introducción de IoT Hub Azure Explorador de datos | Microsoft Docs
description: En este artículo se describe la introducción de IoT Hub en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: c3ed0fa8608f2be5739c1143a648230f792e5d40
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373396"
---
# <a name="ingest-from-iot-hub"></a>Ingesta desde IoT Hub

[Azure IOT Hub](https://docs.microsoft.com/azure/iot-hub/about-iot-hub) es un servicio administrado, hospedado en la nube, que actúa como centro central de mensajes para la comunicación bidireccional entre la aplicación de IOT y los dispositivos que administra. Azure Explorador de datos ofrece una ingesta continua de centros de IoT administrados por el cliente, usando su [punto de conexión integrado compatible con el centro de eventos](https://docs.microsoft.com/azure/iot-hub/iot-hub-devguide-messages-d2c#routing-endpoints).

## <a name="data-format"></a>Formato de datos

* Los datos se leen desde el punto de conexión del centro de eventos en forma de objetos [EventData](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata?view=azure-dotnet) .
* La carga del evento puede estar en uno de los [formatos admitidos por Azure explorador de datos](../../../ingestion-supported-formats.md).

## <a name="ingestion-properties"></a>Propiedades de la ingesta

Propiedades de ingesta indica el proceso de ingesta. Dónde enrutar los datos y cómo procesarlos. Puede especificar [las propiedades de ingesta](../../../ingestion-properties.md) de la ingesta de eventos mediante [EventData. Properties](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata.properties?view=azure-dotnet#Microsoft_ServiceBus_Messaging_EventData_Properties). Puede establecer las siguientes propiedades:

|Propiedad |Descripción|
|---|---|
| Tabla | Nombre (distingue mayúsculas de minúsculas) de la tabla de destino existente. Invalida el valor de `Table` establecido en la hoja `Data Connection`. |
| Formato | Formato de datos. Invalida el valor de `Data format` establecido en la hoja `Data Connection`. |
| IngestionMappingReference | Nombre de la [asignación de ingesta](../create-ingestion-mapping-command.md) existente que se va a usar. Invalida el valor de `Column mapping` establecido en la hoja `Data Connection`.|
| Encoding |  Codificación de datos, el valor predeterminado es UTF8. Puede ser cualquiera de las [codificaciones admitidas por .net](https://docs.microsoft.com/dotnet/api/system.text.encoding?view=netframework-4.8#remarks). |

## <a name="events-routing"></a>Enrutamiento de eventos

Al configurar una conexión de IoT Hub a Azure Explorador de datos Cluster, se especifican las propiedades de la tabla de destino (nombre de tabla, formato de datos y asignación). Este es el enrutamiento predeterminado para los datos, también denominados `static routing` .
También puede especificar las propiedades de la tabla de destino para cada evento, mediante las propiedades del evento. La conexión enrutará dinámicamente los datos tal y como se especifica en [EventData. Properties](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.eventdata.properties?view=azure-dotnet#Microsoft_ServiceBus_Messaging_EventData_Properties), invalidando las propiedades estáticas de este evento.

## <a name="event-system-properties-mapping"></a>Asignación de propiedades del sistema de eventos

Las propiedades del sistema son una colección que se usa para almacenar las propiedades establecidas por el servicio IoT hubs, en el momento en que se recibe el evento. La conexión de Azure Explorador de datos IoT Hub incrustará las propiedades seleccionadas en el desembarque de datos de la tabla.

> [!Note]
> * Para la `csv` asignación, las propiedades se agregan al principio del registro en el orden que se muestra en la tabla siguiente. Para `json` la asignación, las propiedades se agregan según los nombres de propiedad de la tabla siguiente.

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

Si seleccionó **propiedades del sistema de eventos** en la sección **origen de datos** de la tabla, debe incluir las propiedades en el esquema y la asignación de la tabla.

**Ejemplo de esquema de tabla**

Si los datos incluyen tres columnas ( `Timespan` , `Metric` y `Value` ) y las propiedades que se incluyen son `x-opt-enqueued-time` y `x-opt-offset` , cree o modifique el esquema de tabla mediante este comando:

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

## <a name="create-iot-hub-connection"></a>Crear conexión IoT Hub

> [!Note]
> Para obtener el mejor rendimiento, cree todos los recursos en la misma región que el clúster de Explorador de datos de Azure.

### <a name="create-an-iot-hub"></a>Creación de un IoT Hub

Si aún no tiene una, [cree una instancia de IOT Hub](../../../ingest-data-iot-hub.md#create-an-iot-hub).

> [!Note]
> * El `device-to-cloud partitions` recuento no es modificable, por lo que debe tener en cuenta la escala a largo plazo al establecer el número de particiones.
> * El grupo de consumidores *debe* ser uniqe por consumidor. Cree un grupo de consumidores dedicado a la conexión de Kusto. Busque el recurso en Azure portal y vaya a `Built-in endpoints` para agregar un nuevo grupo de consumidores.

### <a name="data-ingestion-connection-to-azure-data-explorer"></a>Conexión de ingesta de datos a Azure Explorador de datos

* Mediante Azure Portal: [Conecte la tabla de explorador de datos de Azure a IOT Hub](../../../ingest-data-iot-hub.md#connect-azure-data-explorer-table-to-iot-hub).
* Uso del SDK de .NET de administración de Azure Explorador de datos: [incorporación de una conexión de datos IOT Hub](../../../data-connection-iot-hub-csharp.md#add-an-iot-hub-data-connection)
* Uso del SDK de Python de Azure Explorador de datos Management: [incorporación de una conexión de datos IOT Hub](../../../data-connection-iot-hub-python.md#add-an-iot-hub-data-connection)
* Con la plantilla de ARM: [Azure Resource Manager plantilla para agregar una conexión de datos de IOT Hub](../../../data-connection-iot-hub-resource-manager.md#azure-resource-manager-template-for-adding-an-iot-hub-data-connection)

> [!Note]
> Si **mis datos incluyen** la información de enrutamiento seleccionada, *debe* proporcionar la información de [enrutamiento](#events-routing) necesaria como parte de las propiedades de eventos.

> [!Note]
> Una vez establecida la conexión, ingesta datos a partir de los eventos que se ponen en cola después de la hora de creación.

### <a name="generating-data"></a>Generación de datos

* Vea el [proyecto de ejemplo](https://github.com/Azure-Samples/azure-iot-samples-csharp/tree/master/iot-hub/Quickstarts/simulated-device) que simula un dispositivo y genera datos.