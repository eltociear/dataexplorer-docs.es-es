---
author: orspod
ms.service: data-explorer
ms.topic: include
ms.date: 02/27/2020
ms.author: orspodek
ms.openlocfilehash: a2297301a0b9c0540c73c0f50483cccfc3181a0f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493408"
---
### <a name="event-system-properties-mapping"></a>Asignación de propiedades del sistema de eventos

> [!Note]
> * Las propiedades del sistema son compatibles con los eventos de registro único.
> * En el caso de la asignación `csv`, las propiedades se agregan al principio del registro. En el caso de la asignación `json`, las propiedades se agregan según el nombre que aparece en la lista desplegable.

Si ha seleccionado **Propiedades del sistema de eventos** en la sección **Origen de datos** de la tabla, debe incluir las siguientes propiedades en el esquema y la asignación de la tabla.

**Ejemplo de esquema de tabla**

Si los datos incluyen tres columnas (`Timespan`, `Metric` y `Value`) y las propiedades que incluye son `x-opt-enqueued-time` y `x-opt-offset`, cree o modifique el esquema de tabla mediante este comando:

```kusto
    .create-merge table TestTable (TimeStamp: datetime, Metric: string, Value: int, EventHubEnqueuedTime:datetime, EventHubOffset:string)
```

**Ejemplo de asignación CSV**

Ejecute los siguientes comandos para agregar datos al principio del registro. Observe los valores ordinales.

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
 
**Ejemplo de asignación JSON**

Los datos se agregan mediante los nombres de las propiedades del sistema tal y como aparecen en la hoja **Conexión de datos** de la lista **Propiedades del sistema de eventos**. Ejecute estos comandos:

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
