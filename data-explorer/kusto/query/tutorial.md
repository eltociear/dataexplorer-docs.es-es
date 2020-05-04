---
title: 'Tutorial: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe el tutorial de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/23/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 90d06064069a17d6b1394701bb4ea72483061b9c
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737613"
---
# <a name="tutorial"></a>Tutorial

::: zone pivot="azuredataexplorer"

La mejor manera de obtener información sobre el lenguaje de consulta de Kusto es examinar algunas consultas sencillas para obtener el "funcionamiento" del lenguaje mediante una [base de datos con algunos datos de ejemplo](https://help.kusto.windows.net/Samples). Las consultas que se muestran en este artículo deben ejecutarse en esa base de datos. En `StormEvents` la tabla de esta base de datos de ejemplo se proporciona información sobre las tormentas que se han producido en el

<!--
  TODO: Provide link to reference data we used originally in StormEvents
-->

<!--
  TODO: A few samples below reference non-existent tables, such as Events and Logs.
        We need to add these tables.
-->

## <a name="count-rows"></a>Contar filas

Nuestra base de datos de ejemplo tiene `StormEvents`una tabla denominada.
Para averiguar lo grande que es, canalizaremos su contenido en un operador que simplemente cuente las filas:

* *Sintaxis:* Una consulta es un origen de datos (normalmente un nombre de tabla), seguido opcionalmente de uno o más pares de caracteres de barra vertical y un operador tabular.

```kusto
StormEvents | count
```

Este es el resultado:

|Count|
|-----|
|59066|
    
[operador Count](./countoperator.md).

## <a name="project-select-a-subset-of-columns"></a>Proyecto: seleccionar un subconjunto de columnas

Use [Project](./projectoperator.md) para seleccionar solo las columnas que desee. Vea el ejemplo siguiente que usa el operador [Project](./projectoperator.md) y [Take](./takeoperator.md) .

## <a name="where-filtering-by-a-boolean-expression"></a>Where: filtrado por una expresión booleana

Vamos a ver solo las `flood`s de `California` en febrero-2007:

```kusto
StormEvents
| where StartTime > datetime(2007-02-01) and StartTime < datetime(2007-03-01)
| where EventType == 'Flood' and State == 'CALIFORNIA'
| project StartTime, EndTime , State , EventType , EpisodeNarrative
```

|StartTime|EndTime|Estado|EventType|EpisodeNarrative|
|---|---|---|---|---|
|2007-02-19 00:00:00.0000000|2007-02-19 08:00:00.0000000|CALIFORNIA|Inundación|Un sistema frontal que se desplaza por el centro de Joaquin del sur de San, ha llevado breves períodos de lluvia intensa a la provincia del Kernismo en las primeras horas de la mañana del 19. La inundación secundaria se comunicó en la autopista de estado 166 cerca de Pérez.|

## <a name="take-show-me-n-rows"></a>Take: mostrarme n filas

Veamos algunos datos: ¿Qué hay en un ejemplo de 5 filas?

```kusto
StormEvents
| take 5
| project  StartTime, EndTime, EventType, State, EventNarrative  
```

|StartTime|EndTime|EventType|Estado|EventNarrative|
|---|---|---|---|---|
|2007-09-18 20:00:00.0000000|2007-09-19 18:00:00.0000000|Lluvia intensa|FLORIDA|En un período de 24 horas a lo largo de las partes del Condado de Volusia litoral.|
|2007-09-20 21:57:00.0000000|2007-09-20 22:05:00.0000000|Tornado|FLORIDA|Un tornado se toca en la ciudad de Eustis en el extremo septentrional del lago torcida occidental. El tornado se ha intensificado rápidamente hasta el nivel de EF1 al pasar el noroeste del norte a través de Eustis. La pista estaba por debajo de dos kilómetros de longitud y tenía un ancho máximo de 300 metros.  El tornado destruyó 7 casas. Veinte 25 casas recibieron daños importantes y 81 casas INLA insignifican daños menores. No hubo lesiones graves y los daños en la propiedad se establecieron en $6,2 millones.|
|2007-09-29 08:11:00.0000000|2007-09-29 08:11:00.0000000|Waterspout|SUR DE ATLÁNTICO|Un waterspout formado en el sudeste de Atlántico de Melbourne Beach y que se ha desplazado brevemente hacia tierra.|
|2007-12-20 07:50:00.0000000|2007-12-20 07:53:00.0000000|Viento de tormenta|MISSISSIPPI|Se han desactivado numerosos árboles grandes con algunas líneas de energía. Se produjeron daños en el Condado de Adams oriental.|
|2007-12-30 16:00:00.0000000|2007-12-30 16:05:00.0000000|Viento de tormenta|Georgia|El envío del Condado informaba de que varios árboles se redujeron a lo largo del bucle Quincey Batten cerca del estado Road 206. Se calculó el costo de eliminación de árboles.|

Pero [se muestran las](./takeoperator.md) filas de la tabla sin ningún orden determinado, así que vamos a ordenarlas.
* [Limit](./takeoperator.md) es un alias para [Take](./takeoperator.md) y tendrá el mismo efecto.

## <a name="sort-and-top"></a>ordenar y superior

* *Sintaxis:* Algunos operadores tienen parámetros introducidos por palabras clave `by`como.
* `desc` = orden descendente, `asc` = orden ascendente.

Muéstrame las primeras n filas, ordenadas por una columna en particular:

```kusto
StormEvents
| top 5 by StartTime desc
| project  StartTime, EndTime, EventType, State, EventNarrative  
```

|StartTime|EndTime|EventType|Estado|EventNarrative|
|---|---|---|---|---|
|2007-12-31 22:30:00.0000000|2007-12-31 23:59:00.0000000|Storm Storm|MICHIGAN|Este evento de nieve pesado continuó en las primeras horas de la mañana del día del año nuevo.|
|2007-12-31 22:30:00.0000000|2007-12-31 23:59:00.0000000|Storm Storm|MICHIGAN|Este evento de nieve pesado continuó en las primeras horas de la mañana del día del año nuevo.|
|2007-12-31 22:30:00.0000000|2007-12-31 23:59:00.0000000|Storm Storm|MICHIGAN|Este evento de nieve pesado continuó en las primeras horas de la mañana del día del año nuevo.|
|2007-12-31 23:53:00.0000000|2007-12-31 23:53:00.0000000|Viento alta|CALIFORNIA|North to Northeast gusting en torno a 58 mph se comunicó en las montañas del Condado de Ventura.|
|2007-12-31 23:53:00.0000000|2007-12-31 23:53:00.0000000|Viento alta|CALIFORNIA|El sensor de RAWS de muelles cálidos comunicó Northerly gusting a 58 mph.|

Lo mismo se puede lograr mediante el uso de [Sort](./sortoperator.md) y después [Take](./takeoperator.md) (operador).

```kusto
StormEvents
| sort by StartTime desc
| take 5
| project  StartTime, EndLat, EventType, EventNarrative
```

## <a name="extend-compute-derived-columns"></a>extender: calcular columnas derivadas

Cree una nueva columna calculando un valor en cada fila:

```kusto
StormEvents
| limit 5
| extend Duration = EndTime - StartTime 
| project StartTime, EndTime, Duration, EventType, State
```

|StartTime|EndTime|Duration|EventType|Estado|
|---|---|---|---|---|
|2007-09-18 20:00:00.0000000|2007-09-19 18:00:00.0000000|22:00:00|Lluvia intensa|FLORIDA|
|2007-09-20 21:57:00.0000000|2007-09-20 22:05:00.0000000|00:08:00|Tornado|FLORIDA|
|2007-09-29 08:11:00.0000000|2007-09-29 08:11:00.0000000|00:00:00|Waterspout|SUR DE ATLÁNTICO|
|2007-12-20 07:50:00.0000000|2007-12-20 07:53:00.0000000|00:03:00|Viento de tormenta|MISSISSIPPI|
|2007-12-30 16:00:00.0000000|2007-12-30 16:05:00.0000000|00:05:00|Viento de tormenta|Georgia|

Es posible volver a usar el nombre de columna y asignar el resultado de cálculo a la misma columna.
Por ejemplo:

```kusto
print x=1
| extend x = x + 1, y = x
| extend x = x + 1
```

|x|y|
|---|---|
|3|1|

Las [expresiones escalares](./scalar-data-types/index.md) pueden incluir todos los operadores`+`habituales `-`( `*`, `/`, `%`,,) y hay una serie de funciones útiles.

## <a name="summarize-aggregate-groups-of-rows"></a>resumir: grupos de filas agregados

Cuente el número de eventos que proceden de cada país:

```kusto
StormEvents
| summarize event_count = count() by State
```

[resumir](./summarizeoperator.md) los grupos en filas que tienen los mismos valores en `by` la cláusula y, a continuación, utiliza la función de `count`agregación (como) para combinar cada grupo en una sola fila. Por lo tanto, en este caso, hay una fila para cada Estado y una columna para el recuento de filas en ese estado.

Hay un intervalo de [funciones de agregación](./summarizeoperator.md#list-of-aggregation-functions), y puede usar varias de ellas en un operador de resumen para generar varias columnas calculadas. Por ejemplo, podríamos obtener el recuento de tormentas en cada Estado y también una suma del tipo único de Storm por estado,  
Luego, podríamos usar [Top](./topoperator.md) para obtener el mayor número de Estados afectados por Storm:

```kusto
StormEvents 
| summarize StormCount = count(), TypeOfStorms = dcount(EventType) by State
| top 5 by StormCount desc
```

|Estado|StormCount|TypeOfStorms|
|---|---|---|
|TEXAS|4701|27|
|KANSAS|3166|21|
|IOWA|2337|19|
|ILLINOIS|2022|23|
|MISSOURI|2016|20|

El resultado de un resumen tiene:

* cada columna con nombre en `by`;
* columna para cada expresión calculada;
* una fila para cada combinación de valores `by` .

## <a name="summarize-by-scalar-values"></a>Resumen por valores escalares

Puede usar valores escalares (numéricos, de tiempo o de intervalo) `by` en la cláusula, pero querrá colocar los valores en ubicaciones.  
La función [bin ()](./binfunction.md) es útil para esto:

```kusto
StormEvents
| where StartTime > datetime(2007-02-14) and StartTime < datetime(2007-02-21)
| summarize event_count = count() by bin(StartTime, 1d)
```

Esto reduce todas las marcas de tiempo a intervalos de 1 día:

|StartTime|event_count|
|---|---|
|2007-02-14 00:00:00.0000000|180|
|2007-02-15 00:00:00.0000000|66|
|2007-02-16 00:00:00.0000000|164|
|2007-02-17 00:00:00.0000000|103|
|2007-02-18 00:00:00.0000000|22|
|2007-02-19 00:00:00.0000000|52|
|2007-02-20 00:00:00.0000000|60|

La [Ubicación ()](./binfunction.md) es la misma que la función [Floor ()](./floorfunction.md) en muchos idiomas. Simplemente reduce cada valor al múltiplo más cercano del módulo que proporciona, de modo que [resumir](./summarizeoperator.md) puede asignar las filas a los grupos.

## <a name="render-display-a-chart-or-table"></a>Render: mostrar un gráfico o una tabla

Proyectar dos columnas y usarlas como eje x e y de un gráfico:

```kusto
StormEvents 
| summarize event_count=count(), mid = avg(BeginLat) by State 
| sort by mid
| where event_count > 1800
| project State, event_count
| render columnchart
```

:::image type="content" source="images/tutorial/event-counts-state.png" alt-text="Gráfico de columnas de los recuentos de eventos de Storm por estado":::

Aunque hemos quitado `mid` en la operación del proyecto, todavía lo necesitamos si queremos que el gráfico muestre los países en ese orden.

En realidad, ' Render ' es una característica del cliente en lugar de parte del lenguaje de consulta. Aún así, se integra en el lenguaje y es muy útil para la previsión de los resultados.


## <a name="timecharts"></a>Gráficos de tiempo

Volviendo a los contenedores numéricos, vamos a mostrar una serie temporal:

```kusto
StormEvents
| summarize event_count=count() by bin(StartTime, 1d)
| render timechart
```

:::image type="content" source="images/tutorial/time-series-start-bin.png" alt-text="Eventos de gráfico de líneas discretizan por hora":::

## <a name="multiple-series"></a>Varias series

Use varios valores en una cláusula `summarize by` para crear una fila independiente para cada combinación de valores:

```kusto
StormEvents 
| where StartTime > datetime(2007-06-04) and StartTime < datetime(2007-06-10) 
| where Source in ("Source","Public","Emergency Manager","Trained Spotter","Law Enforcement")
| summarize count() by bin(StartTime, 10h), Source
```

:::image type="content" source="images/tutorial/table-count-source.png" alt-text="Recuento de tablas por origen":::

Simplemente agregue el término de representación al anterior: `| render timechart`.

:::image type="content" source="images/tutorial/line-count-source.png" alt-text="Recuento de gráficos de líneas por origen":::

Observe que `render timechart` usa la primera columna como eje x y, a continuación, muestra las otras columnas como líneas independientes.

## <a name="daily-average-cycle"></a>Ciclo medio diario

¿Cómo varía la actividad en el día promedio?

Recuento de eventos por el módulo de tiempo un día, discretizan en horas. Tenga en cuenta que `floor` usamos en lugar de bin:

```kusto
StormEvents
| extend hour = floor(StartTime % 1d , 1h)
| summarize event_count=count() by hour
| sort by hour asc
| render timechart
```

:::image type="content" source="images/tutorial/time-count-hour.png" alt-text="Número de gráficos de tiempo por hora":::

Actualmente, `render` no etiqueta las duraciones correctamente, pero podríamos usar `| render columnchart` en su lugar:

:::image type="content" source="images/tutorial/column-count-hour.png" alt-text="Gráfico de columnas recuento por hora":::

## <a name="compare-multiple-daily-series"></a>Comparación de varias series diarias

¿Cómo varía la actividad en la hora del día en distintos Estados?

```kusto
StormEvents
| extend hour= floor( StartTime % 1d , 1h)
| where State in ("GULF OF MEXICO","MAINE","VIRGINIA","WISCONSIN","NORTH DAKOTA","NEW JERSEY","OREGON")
| summarize event_count=count() by hour, State
| render timechart
```

:::image type="content" source="images/tutorial/time-hour-state.png" alt-text="Gráfico de tiempo por hora y estado":::

Divida por `1h` para convertir el eje x en un número de hora en lugar de una duración:

```kusto
StormEvents
| extend hour= floor( StartTime % 1d , 1h)/ 1h
| where State in ("GULF OF MEXICO","MAINE","VIRGINIA","WISCONSIN","NORTH DAKOTA","NEW JERSEY","OREGON")
| summarize event_count=count() by hour, State
| render columnchart
```

:::image type="content" source="images/tutorial/column-hour-state.png" alt-text="Gráfico de columnas por hora y estado":::

## <a name="join"></a>Join

¿Cómo buscar dos EventTypess determinados en qué estado ocurrieron ambos?

Puede extraer eventos de Storm con el primer EventType y con el segundo EventType y después unir los dos conjuntos en el estado.

```kusto
StormEvents
| where EventType == "Lightning"
| join (
    StormEvents 
    | where EventType == "Avalanche"
) on State  
| distinct State
```

:::image type="content" source="images/tutorial/join-events-la.png" alt-text="Eventos de combinación Lightning y avalancha":::

## <a name="user-session-example-of-join"></a>Ejemplo de sesión de usuario de Join

En esta sección no se utiliza `StormEvents` la tabla.

Suponga que tiene datos que incluyen eventos que marcan el inicio y el final de cada sesión de usuario, con un identificador único para cada sesión. 

¿Cuánto dura cada sesión de usuario?

Al usar `extend` para proporcionar un alias para las dos marcas de tiempo, puede calcular la duración de la sesión.

```kusto
Events
| where eventName == "session_started"
| project start_time = timestamp, stop_time, country, session_id
| join ( Events
    | where eventName == "session_ended"
    | project stop_time = timestamp, session_id
    ) on session_id
| extend duration = stop_time - start_time
| project start_time, stop_time, country, duration
| take 10
```

:::image type="content" source="images/tutorial/user-session-extend.png" alt-text="Extensión de la sesión de usuario":::

Es recomendable usar `project` para seleccionar solo las columnas que se necesitan antes de realizar la combinación.
En las mismas cláusulas, cambiamos el nombre de la columna de marca de tiempo.

## <a name="plot-a-distribution"></a>Trazado de una distribución

¿Cuántas tormentas hay de diferentes longitudes?

```kusto
StormEvents
| extend  duration = EndTime - StartTime
| where duration > 0s
| where duration < 3h
| summarize event_count = count()
    by bin(duration, 5m)
| sort by duration asc
| render timechart
```

:::image type="content" source="images/tutorial/event-count-duration.png" alt-text="Recuento de eventos gráfico por duración":::

O use `| render columnchart`:

:::image type="content" source="images/tutorial/column-event-count-duration.png" alt-text="Recuento de eventos de gráfico de columna gráfico por duración":::

## <a name="percentiles"></a>Percentiles

¿Qué intervalos de duración cubren distintos porcentajes de tormentas?

Use la consulta anterior, pero reemplace `render` por:

```kusto
| summarize percentiles(duration, 5, 20, 50, 80, 95)
```

En este caso, no se proporciona `by` ninguna cláusula, por lo que el resultado es una sola fila:

:::image type="content" source="images/tutorial/summarize-percentiles-duration.png" alt-text="Tabla de Resumen de percentiles por duración":::

De lo que podemos ver que:

* el 5% de las Storm tienen una duración inferior a 5 m;
* el 50% de tormentas es inferior a 1 hora.
* el 5% de las tormentas en último al menos 2H 50m.

Para obtener un desglose independiente para cada Estado, solo tenemos que incluir la columna de estado por separado a través de ambos operadores de Resumen:

```kusto
StormEvents
| extend  duration = EndTime - StartTime
| where duration > 0s
| where duration < 3h
| summarize event_count = count()
    by bin(duration, 5m), State
| sort by duration asc
| summarize percentiles(duration, 5, 20, 50, 80, 95) by State
```

:::image type="content" source="images/tutorial/summarize-percentiles-state.png" alt-text="Tabla de Resumen de la duración de los percentiles por estado":::

## <a name="let-assign-a-result-to-a-variable"></a>Let: asignación de un resultado a una variable

Use [Let](./letstatement.md) para separar las partes de la expresión de consulta en el ejemplo anterior de ' join '. Los resultados no cambian:

```kusto
let LightningStorms = 
    StormEvents
    | where EventType == "Lightning";
let AvalancheStorms = 
    StormEvents
    | where EventType == "Avalanche";
LightningStorms 
| join (AvalancheStorms) on State
| distinct State
```

> Sugerencia: en el cliente de Kusto, no incluya líneas en blanco entre las partes de este. Asegúrese de ejecutar todo.

## <a name="combining-data-from-several-databases-in-a-query"></a>Combinar datos de varias bases de datos en una consulta

Consulte [las consultas entre bases de datos](./cross-cluster-or-database-queries.md) para obtener información detallada

Al escribir una consulta del estilo:

```kusto
Logs | where ...
```

La tabla denominada logs debe estar en la base de datos predeterminada. Si desea tener acceso a las tablas desde otra base de datos, use la sintaxis siguiente:

```kusto
database("db").Table
```

Por lo tanto, si tiene bases de datos denominadas *diagnósticos* y *telemetría* y quiere poner en correlación algunos de sus datos, puede escribir (suponiendo que el *diagnóstico* es la base de datos predeterminada)

```kusto
Logs | join database("Telemetry").Metrics on Request MachineId | ...
```

o bien, si la base de datos predeterminada es la *telemetría*

```kusto
union Requests, database("Diagnostics").Logs | ...
```
    
En todo lo anterior se supone que ambas bases de datos residen en el clúster al que está conectado actualmente. Supongamos que la base de datos de *telemetría* pertenecía a otro clúster llamado *TelemetryCluster.kusto.Windows.net* para tener acceso a ella.

```kusto
Logs | join cluster("TelemetryCluster").database("Telemetry").Metrics on Request MachineId | ...
```

> Nota: cuando se especifica el clúster, la base de datos es obligatoria

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end
