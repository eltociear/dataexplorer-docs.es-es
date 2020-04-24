---
title: Control de datos duplicados en Azure Data Explorer
description: En este tema se muestran diversos enfoques para abordar los datos duplicados cuando se usa Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 12/19/2018
ms.openlocfilehash: 681bb931f2ccb4291d7e186024ca89168b579e0a
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493476"
---
# <a name="handle-duplicate-data-in-azure-data-explorer"></a>Control de datos duplicados en Azure Data Explorer

Los dispositivos que envían datos a la nube mantienen una caché local de los datos. Según el tamaño de los datos, la memoria caché local puede almacenar datos de varios días o incluso meses. Es conveniente proteger las bases de datos analíticos de aquellos dispositivos que por error vuelven a enviar los datos almacenados en caché y provocan la duplicación de datos en la base de datos analíticos. En este tema se describen los procedimientos recomendados para controlar los datos duplicados para estos tipos de escenarios.

La mejor solución para la duplicación de datos es evitar dicha duplicación. Si es posible, corrija el problema con antelación en la canalización de datos, lo que ahorrará costos asociados con el movimiento de datos a lo largo de la canalización de datos y evitará que se empleen recursos en tratar con los datos duplicados introducidos en el sistema. Sin embargo, en situaciones en las que no es posible modificar el sistema de origen, hay varias maneras de gestionar este escenario.

## <a name="understand-the-impact-of-duplicate-data"></a>Conocimiento del impacto de los datos duplicados

Supervise el porcentaje de datos duplicados. Una vez detectado el porcentaje de datos duplicados, puede analizar el ámbito del problema y el impacto sobre el negocio para elegir la solución adecuada.

Ejemplo de consulta para identificar el porcentaje de registros duplicados:

```kusto
let _sample = 0.01; // 1% sampling
let _data =
DeviceEventsAll
| where EventDateTime between (datetime('10-01-2018 10:00') .. datetime('10-10-2018 10:00'));
let _totalRecords = toscalar(_data | count);
_data
| where rand()<= _sample
| summarize recordsCount=count() by hash(DeviceId) + hash(EventId) + hash(StationId)  // Use all dimensions that make row unique. Combining hashes can be improved
| summarize duplicateRecords=countif(recordsCount  > 1)
| extend duplicate_percentage = (duplicateRecords / _sample) / _totalRecords  
```

## <a name="solutions-for-handling-duplicate-data"></a>Soluciones para tratar datos duplicados

### <a name="solution-1-dont-remove-duplicate-data"></a>Solución 1: No eliminar los datos duplicados

Determine los requisitos de su negocio y su nivel de tolerancia a los datos duplicados. Algunos conjuntos de datos pueden aceptar un determinado porcentaje de datos duplicados. Si los datos duplicados no tienen una repercusión importante, puede ignorar su presencia. La ventaja de no eliminar los datos duplicados es que no se sobrecargará adicionalmente el proceso de ingesta o el rendimiento de consultas.

### <a name="solution-2-handle-duplicate-rows-during-query"></a>Solución 2: Administrar las filas duplicadas durante la consulta

Otra opción es filtrar las filas duplicadas en los datos durante la consulta. La función de agregado [`arg_max()`](kusto/query/arg-max-aggfunction.md) se puede usar para filtrar los registros duplicados y devolver el último registro en función de la marca de tiempo (u otra columna). La ventaja de este método es una ingesta más rápida, ya que la desduplicación se produce durante el tiempo de consulta. Además, todos los registros (incluidos los duplicados) están disponibles para auditoría y solución de problemas. La desventaja de utilizar la función `arg_max` es el tiempo de consulta adicional y la carga en la CPU cada vez que se consultan los datos. Según la cantidad de datos consultados, esta solución podría no ser funcional o consumir demasiada memoria, por lo que sería necesario recurrir a otras opciones.

En el ejemplo siguiente, se consulta el último registro ingerido para un conjunto de columnas que determinan los registros únicos:

```kusto
DeviceEventsAll
| where EventDateTime > ago(90d)
| summarize hint.strategy=shuffle arg_max(EventDateTime, *) by DeviceId, EventId, StationId
```

Esta consulta también se puede colocar dentro de una función en lugar de consultar directamente la tabla:

```kusto
.create function DeviceEventsView
{
DeviceEventsAll
| where EventDateTime > ago(90d)
| summarize arg_max(EventDateTime, *) by DeviceId, EventId, StationId
}
```

### <a name="solution-3-filter-duplicates-during-the-ingestion-process"></a>Solución 3: Filtrar duplicados durante el proceso de ingesta

Otra solución consiste en filtrar duplicados durante el proceso de ingesta. El sistema ignora los datos duplicados durante la ingesta en tablas Kusto. Los datos se ingieren en una tabla de almacenamiento provisional y se copian en otra tabla después de eliminar las filas duplicadas. La ventaja de esta solución es que el rendimiento de las consultas mejora drásticamente en comparación con la solución anterior. Entre sus desventajas se incluyen un mayor tiempo de ingesta y costos adicionales de almacenamiento de datos. Además, esta solución solo funciona si las réplicas no se ingieren simultáneamente. Si hay varias ingestas simultáneas que contienen registros duplicados, se pueden ingerir todos los registros, ya que el proceso de desduplicación no encontrará ningún registro coincidente existente en la tabla.    

El siguiente ejemplo describe este método:

1. Cree otra tabla con el mismo esquema:

    ```kusto
    .create table DeviceEventsUnique (EventDateTime: datetime, DeviceId: int, EventId: int, StationId: int)
    ```

1. Cree una función para filtrar los registros duplicados mediante la anticombinación de los nuevos registros con los ingeridos anteriormente.

    ```kusto
    .create function RemoveDuplicateDeviceEvents()
    {
    DeviceEventsAll
    | join hint.strategy=broadcast kind = anti
        (
        DeviceEventsUnique
        | where EventDateTime > ago(7d)   // filter the data for certain time frame
        | limit 1000000   //set some limitations (few million records) to avoid choking-up the system during outage recovery

        ) on DeviceId, EventId, StationId
    }
    ```

    > [!NOTE]
    > Las combinaciones son operaciones vinculadas a la CPU y agregan una carga adicional al sistema.

1. Establezca la [directiva de actualización](kusto/management/update-policy.md) en la tabla `DeviceEventsUnique`. La directiva de actualización se activa cuando entran nuevos datos en la tabla `DeviceEventsAll`. El motor de Kusto ejecutará automáticamente la función a medida que se crean nuevas [extensiones](kusto/management/extents-overview.md). El procesamiento se limita a los datos recién creados. El comando siguiente une la tabla de origen (`DeviceEventsAll`), la tabla de destino (`DeviceEventsUnique`) y la función `RemoveDuplicatesDeviceEvents` para crear la directiva de actualización.

    ```kusto
    .alter table DeviceEventsUnique policy update
    @'[{"IsEnabled": true, "Source": "DeviceEventsAll", "Query": "RemoveDuplicateDeviceEvents()", "IsTransactional": true, "PropagateIngestionProperties": true}]'
    ```

    > [!NOTE]
    > La directiva de actualización prolonga la duración de la ingesta, puesto que los datos se filtran durante la ingesta y, a continuación, son ingeridos dos veces (para la tabla `DeviceEventsAll` y para la tabla `DeviceEventsUnique`).

1. (Opcional) Defina una retención de datos inferior en la tabla `DeviceEventsAll` para evitar almacenar copias de los datos. Seleccione el número de días según el volumen de datos y el período de tiempo que desee retener los datos para la solución de problemas. Puede definir la retención en `0d` días para ahorrar costos de bienes vendidos (COGS) y mejorar el rendimiento, ya que los datos no se cargan en el almacenamiento.

    ```kusto
    .alter-merge table DeviceEventsAll policy retention softdelete = 1d
    ```

## <a name="summary"></a>Resumen

La duplicación de datos se puede tratar de varias maneras. Evalúe las opciones detenidamente, teniendo en cuenta el precio y el rendimiento, para determinar el método correcto para su negocio.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Write queries for Azure Data Explorer](write-queries.md) (Escritura de consultas del Explorador de datos de Azure)
