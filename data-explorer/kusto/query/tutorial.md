---
title: Tutorial - Explorador de Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe Tutorial en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/23/2020
zone_pivot_group_filename: kusto/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: f8a5a7b8f96fdd75ccff5dcfe8499ab98a2d4380
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663332"
---
# <a name="tutorial"></a>Tutorial

::: zone pivot="azuredataexplorer"

La mejor manera de aprender sobre el lenguaje de consulta Kusto es mirar algunas consultas simples para obtener la "sensación" del lenguaje utilizando una base de datos con algunos datos de [ejemplo.](https://help.kusto.windows.net/Samples) Las consultas que se muestran en este artículo deben ejecutarse en esa base de datos. La `StormEvents` tabla de esta base de datos de ejemplo proporciona información sobre las tormentas que ocurrieron en los EE. UU.

<!--
  TODO: Provide link to reference data we used originally in StormEvents
-->

<!--
  TODO: A few samples below reference non-existent tables, such as Events and Logs.
        We need to add these tables.
-->

## <a name="count-rows"></a>Contar filas

Nuestra base de datos `StormEvents`de ejemplo tiene una tabla llamada .
Para averiguar qué tan grande es, canalizaremos su contenido en un operador que simplemente cuenta las filas:

* *Sintaxis:* Una consulta es un origen de datos (normalmente un nombre de tabla), opcionalmente seguido de uno o más pares del carácter de canalización y algún operador tabular.

```kusto
StormEvents | count
```

Este es el resultado:

|Count|
|-----|
|59066|
    
[operador de recuento](./countoperator.md).

## <a name="project-select-a-subset-of-columns"></a>proyecto: seleccione un subconjunto de columnas

Utilice [el proyecto](./projectoperator.md) para seleccionar solo las columnas que desee. Vea el ejemplo siguiente que utiliza el [proyecto](./projectoperator.md) y el operador [take.](./takeoperator.md)

## <a name="where-filtering-by-a-boolean-expression"></a>donde: filtrado por una expresión booleana

Vamos a ver `flood`sólo `California` el s en durante febrero-2007:

```kusto
StormEvents
| where StartTime > datetime(2007-02-01) and StartTime < datetime(2007-03-01)
| where EventType == 'Flood' and State == 'CALIFORNIA'
| project StartTime, EndTime , State , EventType , EpisodeNarrative
```

|StartTime|EndTime|State|EventType|EpisodioNarración|
|---|---|---|---|---|
|2007-02-19 00:00:00.0000000|2007-02-19 08:00:00.0000000|California|Inundación|Un sistema frontal que se mueve a través del sur del valle de San Joaquín trajo breves períodos de lluvia fuerte al oeste del condado de Kern en las primeras horas de la mañana del día 19. Se reportaron inundaciones menores a través de la carretera estatal 166 cerca de Taft.|

## <a name="take-show-me-n-rows"></a>tomar: muéstrame n filas

Veamos algunos datos: ¿Qué hay en un ejemplo de 5 filas?

```kusto
StormEvents
| take 5
| project  StartTime, EndTime, EventType, State, EventNarrative  
```

|StartTime|EndTime|EventType|State|EventNarrative|
|---|---|---|---|---|
|2007-09-18 20:00:00.0000000|2007-09-19 18:00:00.0000000|Lluvia fuerte|Florida|Hasta 9 pulgadas de lluvia cayeron en un período de 24 horas a través de partes del condado costero de Volusia.|
|2007-09-20 21:57:00.0000000|2007-09-20 22:05:00.0000000|Tornado|Florida|Un tornado aterrizó en la ciudad de Eustis, en el extremo norte del lago torcido oeste. El tornado se intensificó rápidamente a la fuerza de EF1 a medida que se movía hacia el noroeste a través de Eustis. La pista tenía poco menos de dos millas de largo y tenía un ancho máximo de 300 yardas.  El tornado destruyó 7 casas. Veintisiete viviendas recibieron daños importantes y 81 viviendas reportaron daños menores. No hubo lesiones graves y los daños a la propiedad se fijaron en 6,2 millones de dólares.|
|2007-09-29 08:11:00.0000000|2007-09-29 08:11:00.0000000|Waterspout|ATLANTIC SOUTH|Un chorro de agua se formó en el Atlántico al sureste de Melbourne Beach y se trasladó brevemente hacia la costa.|
|2007-12-20 07:50:00.0000000|2007-12-20 07:53:00.0000000|Viento de tormenta|Mississippi|Numerosos árboles grandes fueron derribados con algunos abajo en las líneas eléctricas. Los daños ocurrieron en el este del condado de Adams.|
|2007-12-30 16:00:00.0000000|2007-12-30 16:05:00.0000000|Viento de tormenta|Georgia|El envío del condado informó que varios árboles fueron derribados a lo largo de Quincey Batten Loop cerca de State Road 206. Se estimó el costo de la remoción de árboles.|

Pero [tomar](./takeoperator.md) muestra filas de la tabla en ningún orden en particular, así que vamos a ordenarlas.
* [límite](./takeoperator.md) es un alias para [tomar](./takeoperator.md) y tendrá el mismo efecto.

## <a name="sort-and-top"></a>clasificación y la parte superior

* *Sintaxis:* Algunos operadores tienen parámetros introducidos por palabras clave como `by`.
* `desc` = orden descendente, `asc` = orden ascendente.

Muéstrame las primeras n filas, ordenadas por una columna en particular:

```kusto
StormEvents
| top 5 by StartTime desc
| project  StartTime, EndTime, EventType, State, EventNarrative  
```

|StartTime|EndTime|EventType|State|EventNarrative|
|---|---|---|---|---|
|2007-12-31 22:30:00.0000000|2007-12-31 23:59:00.0000000|Tormenta de invierno|MICHIGAN|Este fuerte evento de nieve continuó hasta las primeras horas de la mañana en el día de Año Nuevo.|
|2007-12-31 22:30:00.0000000|2007-12-31 23:59:00.0000000|Tormenta de invierno|MICHIGAN|Este fuerte evento de nieve continuó hasta las primeras horas de la mañana en el día de Año Nuevo.|
|2007-12-31 22:30:00.0000000|2007-12-31 23:59:00.0000000|Tormenta de invierno|MICHIGAN|Este fuerte evento de nieve continuó hasta las primeras horas de la mañana en el día de Año Nuevo.|
|2007-12-31 23:53:00.0000000|2007-12-31 23:53:00.0000000|Viento alto|California|Los vientos de norte a noreste que brotan a alrededor de 58 mph fueron reportados en las montañas del condado de Ventura.|
|2007-12-31 23:53:00.0000000|2007-12-31 23:53:00.0000000|Viento alto|California|El sensor RAWS de Warm Springs reportó vientos del norte que brotan a 58 mph.|

Lo mismo se puede lograr mediante el uso de [la ordenación](./sortoperator.md) y luego [tomar](./takeoperator.md) el operador

```kusto
StormEvents
| sort by StartTime desc
| take 5
| project  StartTime, EndLat, EventType, EventNarrative
```

## <a name="extend-compute-derived-columns"></a>extend: calcular columnas derivadas

Cree una nueva columna calculando un valor en cada fila:

```kusto
StormEvents
| limit 5
| extend Duration = EndTime - StartTime 
| project StartTime, EndTime, Duration, EventType, State
```

|StartTime|EndTime|Duration|EventType|State|
|---|---|---|---|---|
|2007-09-18 20:00:00.0000000|2007-09-19 18:00:00.0000000|22:00:00|Lluvia fuerte|Florida|
|2007-09-20 21:57:00.0000000|2007-09-20 22:05:00.0000000|00:08:00|Tornado|Florida|
|2007-09-29 08:11:00.0000000|2007-09-29 08:11:00.0000000|00:00:00|Waterspout|ATLANTIC SOUTH|
|2007-12-20 07:50:00.0000000|2007-12-20 07:53:00.0000000|00:03:00|Viento de tormenta|Mississippi|
|2007-12-30 16:00:00.0000000|2007-12-30 16:05:00.0000000|00:05:00|Viento de tormenta|Georgia|

Es posible reutilizar el nombre de columna y asignar el resultado del cálculo a la misma columna.
Por ejemplo:

```kusto
print x=1
| extend x = x + 1, y = x
| extend x = x + 1
```

|x|y|
|---|---|
|3|1|

[Las expresiones escalares](./scalar-data-types/index.md) pueden incluir`+` `-`todos `*` `/`los `%`operadores habituales ( , , , , , ), y hay una gama de funciones útiles.

## <a name="summarize-aggregate-groups-of-rows"></a>resumir: grupos agregados de filas

Cuente cuántos eventos provienen de cada país:

```kusto
StormEvents
| summarize event_count = count() by State
```

[resumir](./summarizeoperator.md) grupos juntos filas que `by` tienen los mismos valores en `count`la cláusula y, a continuación, usa la función de agregación (como ) para combinar cada grupo en una sola fila. Por lo tanto, en este caso, hay una fila para cada estado y una columna para el recuento de filas en ese estado.

Hay un rango de funciones de [agregación](./summarizeoperator.md#list-of-aggregation-functions)y puede usar varias de ellas en un operador de resumen para producir varias columnas calculadas. Por ejemplo, podríamos obtener el recuento de tormentas en cada estado y también una suma del tipo único de tormentas por estado,  
entonces podríamos usar [la parte superior](./topoperator.md) para obtener los estados más afectados por las tormentas:

```kusto
StormEvents 
| summarize StormCount = count(), TypeOfStorms = dcount(EventType) by State
| top 5 by StormCount desc
```

|State|StormCount|TypeOfStorms|
|---|---|---|
|TEXAS|4701|27|
|Kansas|3166|21|
|IOWA|2337|19|
|Illinois|2022|23|
|Missouri|2016|20|

El resultado de un resumen tiene:

* cada columna con nombre en `by`;
* una columna para cada expresión calculada;
* una fila para cada combinación de valores `by` .

## <a name="summarize-by-scalar-values"></a>Resumen por valores escalares

Puede usar valores escalares (numéricos, de tiempo `by` o de intervalo) en la cláusula, pero querrá colocar los valores en bins.  
La función [bin()](./binfunction.md) es útil para esto:

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

El [bin()](./binfunction.md) es el mismo que la función [floor()](./floorfunction.md) en muchos idiomas. Simplemente reduce cada valor al múltiplo más cercano del módulo que proporcione, de modo que [summarize](./summarizeoperator.md) pueda asignar las filas a los grupos.

## <a name="render-display-a-chart-or-table"></a>Procesar: muestra un gráfico o una tabla

Proyecte dos columnas y utilícelas como los ejes x e y de un gráfico:

```kusto
StormEvents 
| summarize event_count=count(), mid = avg(BeginLat) by State 
| sort by mid
| where event_count > 1800
| project State, event_count
| render columnchart
```

:::image type="content" source="images/tour/060.png" alt-text="060":::

Aunque eliminamos `mid` en la operación del proyecto, todavía lo necesitamos si queremos que el gráfico muestre los países en ese orden.

Estrictamente hablando, 'render' es una característica del cliente en lugar de parte del lenguaje de consulta. Aún así, está integrado en el lenguaje y es muy útil para imaginar sus resultados.


## <a name="timecharts"></a>Gráficos de tiempo

Volviendo a las bandejas numéricas, vamos a mostrar una serie temporal:

```kusto
StormEvents
| summarize event_count=count() by bin(StartTime, 1d)
| render timechart
```

:::image type="content" source="images/tour/080.png" alt-text="080":::

## <a name="multiple-series"></a>Varias series

Use varios valores en una cláusula `summarize by` para crear una fila independiente para cada combinación de valores:

```kusto
StormEvents 
| where StartTime > datetime(2007-06-04) and StartTime < datetime(2007-06-10) 
| where Source in ("Source","Public","Emergency Manager","Trained Spotter","Law Enforcement")
| summarize count() by bin(StartTime, 10h), Source
```

:::image type="content" source="images/tour/090.png" alt-text="090":::

Simplemente agregue el término render `| render timechart`a lo anterior: .

:::image type="content" source="images/tour/100.png" alt-text="100":::

Observe `render timechart` que utiliza la primera columna como el eje X y, a continuación, muestra las otras columnas como líneas independientes.

## <a name="daily-average-cycle"></a>Ciclo medio diario

¿Cómo varía la actividad con el día promedio?

Contar eventos por el tiempo modulo un día, binned en horas. Tenga en `floor` cuenta que usamos en lugar de bin:

```kusto
StormEvents
| extend hour = floor(StartTime % 1d , 1h)
| summarize event_count=count() by hour
| sort by hour asc
| render timechart
```

:::image type="content" source="images/tour/120.png" alt-text="120":::

Actualmente, `render` no etiqueta las duraciones correctamente, pero podríamos usar `| render columnchart` en su lugar:

:::image type="content" source="images/tour/110.png" alt-text="110":::

## <a name="compare-multiple-daily-series"></a>Comparación de varias series diarias

¿Cómo varía la actividad con la hora del día en diferentes estados?

```kusto
StormEvents
| extend hour= floor( StartTime % 1d , 1h)
| where State in ("GULF OF MEXICO","MAINE","VIRGINIA","WISCONSIN","NORTH DAKOTA","NEW JERSEY","OREGON")
| summarize event_count=count() by hour, State
| render timechart
```

:::image type="content" source="images/tour/130.png" alt-text="130":::

Divida `1h` por para convertir el eje X en número de hora en lugar de una duración:

```kusto
StormEvents
| extend hour= floor( StartTime % 1d , 1h)/ 1h
| where State in ("GULF OF MEXICO","MAINE","VIRGINIA","WISCONSIN","NORTH DAKOTA","NEW JERSEY","OREGON")
| summarize event_count=count() by hour, State
| render columnchart
```

:::image type="content" source="images/tour/140.png" alt-text="140":::

## <a name="join"></a>join

¿Cómo encontrar para dos EventTypes dados en qué estado ambos sucedieron?

Puede extraer eventos de tormenta con el primer EventType y con el segundo EventType y, a continuación, unirlos a los dos conjuntos en State.

```kusto
StormEvents
| where EventType == "Lightning"
| join (
    StormEvents 
    | where EventType == "Avalanche"
) on State  
| distinct State
```

:::image type="content" source="images/tour/145.png" alt-text="145":::

## <a name="user-session-example-of-join"></a>Ejemplo de sesión de usuario de join

Esta sección no `StormEvents` utiliza la tabla.

Supongamos que tiene datos que incluyen eventos que marcan el inicio y el final de cada sesión de usuario, con un identificador único para cada sesión. 

¿Cuánto dura cada sesión de usuario?

Mediante `extend` el uso de proporcionar un alias para las dos marcas de tiempo, puede calcular la duración de la sesión.

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

:::image type="content" source="images/tour/150.png" alt-text="150":::

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

:::image type="content" source="images/tour/170.png" alt-text="170":::

O `| render columnchart`utilizar :

:::image type="content" source="images/tour/160.png" alt-text="160":::

## <a name="percentiles"></a>Percentiles

¿Qué rangos de duraciones cubren diferentes porcentajes de tormentas?

Utilice la consulta anterior, pero reemplace `render` por:

```kusto
| summarize percentiles(duration, 5, 20, 50, 80, 95)
```

En este caso, `by` no proporcionamos ninguna cláusula, por lo que el resultado es una sola fila:

:::image type="content" source="images/tour/180.png" alt-text="180":::

De lo que podemos ver que:

* El 5% de las tormentas tienen una duración de menos de 5 m;
* El 50% de las tormentas duran menos de 1h 25m;
* 5% de las tormentas duran al menos 2h 50m.

Para obtener un desglose independiente para cada estado, solo tenemos que traer la columna de estado por separado a través de ambos operadores de resumen:

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

:::image type="content" source="images/tour/190.png" alt-text="190":::

## <a name="let-assign-a-result-to-a-variable"></a>Let: asignación de un resultado a una variable

Utilice [let](./letstatement.md) para separar las partes de la expresión de consulta en el ejemplo 'join' anterior. Los resultados no cambian:

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

> Consejo: En el cliente de Kusto, no ponga líneas en blanco entre las partes de esto. Asegúrese de ejecutar todo.

## <a name="combining-data-from-several-databases-in-a-query"></a>Combinación de datos de varias bases de datos en una consulta

Consulte [las consultas entre bases de datos](./cross-cluster-or-database-queries.md) para obtener una discusión detallada

Al escribir una consulta del estilo:

```kusto
Logs | where ...
```

La tabla denominada Registros debe estar en la base de datos predeterminada. Si desea tener acceso a tablas desde otra base de datos, utilice la sintaxis siguiente:

```kusto
database("db").Table
```

Por lo tanto, si tiene bases de datos *denominadas Diagnóstico* y *telemetría* y desea correlacionar algunos de sus datos, puede escribir (suponiendo que *Diagnostics* sea la base de datos predeterminada)

```kusto
Logs | join database("Telemetry").Metrics on Request MachineId | ...
```

o si su base de datos predeterminada es *Telemetría*

```kusto
union Requests, database("Diagnostics").Logs | ...
```
    
Todo lo anterior suponía que ambas bases de datos residen en el clúster al que está conectado actualmente. Supongamos que la base de datos de *telemetría* pertenecía a otro clúster denominado *TelemetryCluster.kusto.windows.net,* a continuación, para tener acceso a ella, necesitará

```kusto
Logs | join cluster("TelemetryCluster").database("Telemetry").Metrics on Request MachineId | ...
```

> Nota: cuando se especifica el clúster, la base de datos es obligatoria

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end