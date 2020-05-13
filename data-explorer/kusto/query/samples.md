---
title: 'Ejemplos: Azure Explorador de datos'
description: En este artículo se describen ejemplos de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: fe44323dabb246438f18c9ab01eec0008ad4fe97
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83372961"
---
# <a name="samples"></a>Ejemplos

A continuación se muestran algunas necesidades comunes de consultas y cómo se puede usar el lenguaje de consulta Kusto para cumplirlas.

## <a name="display-a-column-chart"></a>Mostrar un gráfico de columnas

Proyectar dos o más columnas y usarlas como eje x e y de un gráfico:

<!-- csl: https://help.kusto.windows.net/Samples  -->
```kusto 
StormEvents
| where isnotempty(EndLocation) 
| summarize event_count=count() by EndLocation
| top 10 by event_count
| render columnchart
```

* La primera columna forma el eje x. Puede ser Numeric, DateTime o String. 
* Use `where` `summarize` y `top` para limitar el volumen de datos que se muestran.
* Ordene los resultados para definir el orden del eje x.

:::image type="content" source="images/samples/060.png" alt-text="060":::

<a name="activities"></a>
## <a name="get-sessions-from-start-and-stop-events"></a>Obtención de sesiones a partir de eventos de inicio y detención

Supongamos que tenemos un registro de eventos en el que algunos eventos marcan el inicio o el final de una actividad o sesión ampliada. 

|Nombre|City|SessionId|Timestamp|
|---|---|---|---|
|Start|London|2817330|2015-12-09T10:12:02.32|
|Juego|London|2817330|2015-12-09T10:12:52.45|
|Start|Manchester|4267667|2015-12-09T10:14:02.23|
|Stop|London|2817330|2015-12-09T10:23:43.18|
|Cancelar|Manchester|4267667|2015-12-09T10:27:26.29|
|Stop|Manchester|4267667|2015-12-09T10:28:31.72|

Cada evento tiene un identificador de sesión; así pues, el problema radica en hacer coincidir los eventos de inicio y detención que tengan el mismo identificador.

```kusto
let Events = MyLogTable | where ... ;

Events
| where Name == "Start"
| project Name, City, SessionId, StartTime=timestamp
| join (Events 
        | where Name="Stop"
        | project StopTime=timestamp, SessionId) 
    on SessionId
| project City, SessionId, StartTime, StopTime, Duration = StopTime - StartTime
```

Use [`let`](./letstatement.md) para asignar un nombre a una proyección de la tabla que se reduce lo más lejos posible antes de entrar en la combinación.
[`Project`](./projectoperator.md)se utiliza para cambiar los nombres de las marcas de tiempo de modo que las horas de inicio y detención puedan aparecer en el resultado. También selecciona las otras columnas que se quieren ver en el resultado. [`join`](./joinoperator.md)coincide con las entradas de inicio y detención de la misma actividad, creando una fila para cada actividad. Por último, `project` agrega de nuevo una columna para mostrar la duración de la actividad.


|City|SessionId|StartTime|StopTime|Duration|
|---|---|---|---|---|
|London|2817330|2015-12-09T10:12:02.32|2015-12-09T10:23:43.18|00:11:40.46|
|Manchester|4267667|2015-12-09T10:14:02.23|2015-12-09T10:28:31.72|00:14:29.49|

### <a name="get-sessions-without-session-id"></a>Obtener las sesiones, sin los identificadores de sesión

Ahora vamos a suponer que los eventos de inicio y detención no cuentan con un práctico identificador de sesión que podamos hacer coincidir. Sin embargo, tenemos una dirección IP del cliente donde tuvo lugar la sesión. Suponiendo que cada en dirección de cliente se realizó únicamente una sesión a la vez, podemos hacer coincidir cada evento de inicio con el siguiente evento de detención desde la misma dirección IP.

```kusto
Events 
| where Name == "Start" 
| project City, ClientIp, StartTime = timestamp
| join  kind=inner
    (Events
    | where Name == "Stop" 
    | project StopTime = timestamp, ClientIp)
    on ClientIp
| extend duration = StopTime - StartTime 
    // Remove matches with earlier stops:
| where  duration > 0  
    // Pick out the earliest stop for each start and client:
| summarize arg_min(duration, *) by bin(StartTime,1s), ClientIp
```

La combinación unirá cada hora de inicio con todas las horas de detención desde la misma dirección IP de cliente. Por tanto, en primer lugar quitamos las coincidencias con horas de detención anteriores.

A continuación, agrupamos por hora de inicio e IP para obtener un grupo para cada sesión. Se debe proporcionar una `bin` función para el parámetro startTime: Si no es así, Kusto usará automáticamente las ubicaciones de 1 hora, que coincidirán con algunas horas de inicio con horas de detención equivocadas.

`arg_min`selecciona la fila con la menor duración en cada grupo y el `*` parámetro pasa por todas las demás columnas, aunque antepone "min_" a cada nombre de columna. 

:::image type="content" source="images/samples/040.png" alt-text="040"::: 

A continuación, podemos agregar código para contar las duraciones en ubicaciones de tamaño cómodo. Tenemos una ligera preferencia para un gráfico de barras, por lo que dividimos por `1s` para convertir el intervalos en números. 


      // Count the frequency of each duration:
    | summarize count() by duration=bin(min_duration/1s, 10) 
      // Cut off the long tail:
    | where duration < 300
      // Display in a bar chart:
    | sort by duration asc | render barchart 

:::image type="content" source="images/samples/050.png" alt-text="050":::

### <a name="real-example"></a>Ejemplo real

```kusto
Logs  
| filter ActivityId == "ActivityId with Blablabla" 
| summarize max(Timestamp), min(Timestamp)  
| extend Duration = max_Timestamp - min_Timestamp 
 
wabitrace  
| filter Timestamp >= datetime(2015-01-12 11:00:00Z)  
| filter Timestamp < datetime(2015-01-12 13:00:00Z)  
| filter EventText like "NotifyHadoopApplicationJobPerformanceCounters"      
| extend Tenant = extract("tenantName=([^,]+),", 1, EventText) 
| extend Environment = extract("environmentName=([^,]+),", 1, EventText)  
| extend UnitOfWorkId = extract("unitOfWorkId=([^,]+),", 1, EventText)  
| extend TotalLaunchedMaps = extract("totalLaunchedMaps=([^,]+),", 1, EventText, typeof(real))  
| extend MapsSeconds = extract("mapsMilliseconds=([^,]+),", 1, EventText, typeof(real)) / 1000 
| extend TotalMapsSeconds = MapsSeconds  / TotalLaunchedMaps 
| filter Tenant == 'DevDiv' and Environment == 'RollupDev2'  
| filter TotalLaunchedMaps > 0 
| summarize sum(TotalMapsSeconds) by UnitOfWorkId  
| extend JobMapsSeconds = sum_TotalMapsSeconds * 1 
| project UnitOfWorkId, JobMapsSeconds 
| join ( 
wabitrace  
| filter Timestamp >= datetime(2015-01-12 11:00:00Z)  
| filter Timestamp < datetime(2015-01-12 13:00:00Z)  
| filter EventText like "NotifyHadoopApplicationJobPerformanceCounters"  
| extend Tenant = extract("tenantName=([^,]+),", 1, EventText) 
| extend Environment = extract("environmentName=([^,]+),", 1, EventText)  
| extend UnitOfWorkId = extract("unitOfWorkId=([^,]+),", 1, EventText)   
| extend TotalLaunchedReducers = extract("totalLaunchedReducers=([^,]+),", 1, EventText, typeof(real)) 
| extend ReducesSeconds = extract("reducesMilliseconds=([^,]+)", 1, EventText, typeof(real)) / 1000 
| extend TotalReducesSeconds = ReducesSeconds / TotalLaunchedReducers 
| filter Tenant == 'DevDiv' and Environment == 'RollupDev2'  
| filter TotalLaunchedReducers > 0 
| summarize sum(TotalReducesSeconds) by UnitOfWorkId  
| extend JobReducesSeconds = sum_TotalReducesSeconds * 1 
| project UnitOfWorkId, JobReducesSeconds ) 
on UnitOfWorkId 
| join ( 
wabitrace  
| filter Timestamp >= datetime(2015-01-12 11:00:00Z)  
| filter Timestamp < datetime(2015-01-12 13:00:00Z)  
| filter EventText like "NotifyHadoopApplicationJobPerformanceCounters"  
| extend Tenant = extract("tenantName=([^,]+),", 1, EventText) 
| extend Environment = extract("environmentName=([^,]+),", 1, EventText)  
| extend JobName = extract("jobName=([^,]+),", 1, EventText)  
| extend StepName = extract("stepName=([^,]+),", 1, EventText)  
| extend UnitOfWorkId = extract("unitOfWorkId=([^,]+),", 1, EventText)  
| extend LaunchTime = extract("launchTime=([^,]+),", 1, EventText, typeof(datetime))  
| extend FinishTime = extract("finishTime=([^,]+),", 1, EventText, typeof(datetime)) 
| extend TotalLaunchedMaps = extract("totalLaunchedMaps=([^,]+),", 1, EventText, typeof(real))  
| extend TotalLaunchedReducers = extract("totalLaunchedReducers=([^,]+),", 1, EventText, typeof(real)) 
| extend MapsSeconds = extract("mapsMilliseconds=([^,]+),", 1, EventText, typeof(real)) / 1000 
| extend ReducesSeconds = extract("reducesMilliseconds=([^,]+)", 1, EventText, typeof(real)) / 1000 
| extend TotalMapsSeconds = MapsSeconds  / TotalLaunchedMaps  
| extend TotalReducesSeconds = (ReducesSeconds / TotalLaunchedReducers / ReducesSeconds) * ReducesSeconds  
| extend CalculatedDuration = (TotalMapsSeconds + TotalReducesSeconds) * time(1s) 
| filter Tenant == 'DevDiv' and Environment == 'RollupDev2') 
on UnitOfWorkId 
| extend MapsFactor = TotalMapsSeconds / JobMapsSeconds 
| extend ReducesFactor = TotalReducesSeconds / JobReducesSeconds 
| extend CurrentLoad = 1536 + (768 * TotalLaunchedMaps) + (1536 * TotalLaunchedMaps) 
| extend NormalizedLoad = 1536 + (768 * TotalLaunchedMaps * MapsFactor) + (1536 * TotalLaunchedMaps * ReducesFactor) 
| summarize sum(CurrentLoad), sum(NormalizedLoad) by  JobName  
| extend SaveFactor = sum_NormalizedLoad / sum_CurrentLoad 
```

<a name="concurrent-activities"><a/>
## <a name="chart-concurrent-sessions-over-time"></a>Representación de sesiones simultáneas a lo largo del tiempo

Supongamos que tenemos una tabla de actividades con sus horas de inicio y finalización.  Nos gustaría ver un gráfico en el que se refleje el paso del tiempo que muestre cuántas se ejecutan simultáneamente en cualquier momento.

Esta es una entrada de ejemplo, a la que llamaremos `X`:

|SessionId | StartTime | StopTime |
|---|---|---|
| a | 10:01:03 | 10:10:08 |
| b | 10:01:29 | 10:03:10 |
| c | 10:03:02 | 10:05:20 |

Queremos un gráfico en intervalos de 1 minuto, así que deseamos crear algo que, en cada uno de esos intervalos, podamos contar para cada actividad en ejecución.

Este es un resultado intermedio:

```kusto
X | extend samples = range(bin(StartTime, 1m), StopTime, 1m)
```

`range` genera una matriz de valores en los intervalos especificados:

|SessionId | StartTime | StopTime  | ejemplos|
|---|---|---|---|
| a | 10:01:33 | 10:06:31 | [10:01:00,10:02:00,...10:06:00]|
| b | 10:02:29 | 10:03:45 | [10:02:00,10:03:00]|
| c | 10:03:12 | 10:04:30 | [10:03:00,10:04:00]|

Pero en lugar de mantener esas matrices, las expandiremos con [MV-Expand](./mvexpandoperator.md):

```kusto
X | mv-expand samples = range(bin(StartTime, 1m), StopTime , 1m)
```

|SessionId | StartTime | StopTime  | ejemplos|
|---|---|---|---|
| a | 10:01:33 | 10:06:31 | 10:01:00|
| a | 10:01:33 | 10:06:31 | 10:02:00|
| a | 10:01:33 | 10:06:31 | 10:03:00|
| a | 10:01:33 | 10:06:31 | 10:04:00|
| a | 10:01:33 | 10:06:31 | 10:05:00|
| a | 10:01:33 | 10:06:31 | 10:06:00|
| b | 10:02:29 | 10:03:45 | 10:02:00|
| b | 10:02:29 | 10:03:45 | 10:03:00|
| c | 10:03:12 | 10:04:30 | 10:03:00|
| c | 10:03:12 | 10:04:30 | 10:04:00|

Ahora podemos agruparlas por cada una de las horas de ejemplo, contando las veces que aparece cada actividad:

```kusto
X
| mv-expand samples = range(bin(StartTime, 1m), StopTime , 1m)
| summarize count(SessionId) by bin(todatetime(samples),1m)
```

* Necesitamos ToDateTime () porque [MV-Expand](./mvexpandoperator.md) produce una columna de tipo dinámico.
* Necesitamos bin() porque, en el caso de los valores numéricos y de fecha, la función de resumir aplica siempre una función de intervalo con un valor predeterminado si no especifica ninguno. 


| count_SessionId | ejemplos|
|---|---|
| 1 | 10:01:00|
| 2 | 10:02:00|
| 3 | 10:03:00|
| 2 | 10:04:00|
| 1 | 10:05:00|
| 1 | 10:06:00|

Esto se puede representar como un gráfico de barras o un gráfico de tiempo.

## <a name="introduce-null-bins-into-summarize"></a>Introducir las bandejas nulas en resumir

Cuando el `summarize` operador se aplica a través de una clave de grupo formada por una `datetime` columna, normalmente se "bandeja" esos valores en las bandejas de ancho fijo. Por ejemplo:

```kusto
let StartTime=ago(12h);
let StopTime=now()
T
| where Timestamp > StartTime and Timestamp <= StopTime 
| where ...
| summarize Count=count() by bin(Timestamp, 5m)
```

Esta operación genera una tabla con una sola fila por cada grupo de filas de `T` que se encuentran en cada bin de cinco minutos. Lo que no hace es agregar "bandejas nulas": filas para los valores de las ubicaciones de tiempo entre `StartTime` y `StopTime` para las que no hay ninguna fila correspondiente en `T` . 

A menudo, se desea "rellenar" la tabla con esas bandejas. Esta es una manera de hacerlo:

```kusto
let StartTime=ago(12h);
let StopTime=now()
T
| where Timestamp > StartTime and Timestamp <= StopTime 
| summarize Count=count() by bin(Timestamp, 5m)
| where ...
| union ( // 1
  range x from 1 to 1 step 1 // 2
  | mv-expand Timestamp=range(StartTime, StopTime, 5m) to typeof(datetime) // 3
  | extend Count=0 // 4
  )
| summarize Count=sum(Count) by bin(Timestamp, 5m) // 5 
```

Esta es una explicación paso a paso de la consulta anterior: 

1. El uso del `union` operador nos permite agregar más filas a una tabla. Dichas filas se generan mediante la expresión en `union` .
2. Usar el `range` operador para generar una tabla con una sola fila o columna.
   La tabla no se utiliza para ningún elemento que no sea para `mv-expand` trabajar con.
3. Usar el `mv-expand` operador sobre la `range` función para crear tantas filas como una bandeja de 5 minutos entre `StartTime` y `EndTime` .
4. Todo con un `Count` de `0` .
5. Por último, usamos el `summarize` operador para agrupar las ubicaciones del argumento original (izquierda o exterior) en y las `union` ubicaciones del argumento interno en él (es decir, las filas de la ubicación nula). Esto garantiza que la salida tenga una fila por bin, cuyo valor sea cero o el recuento original.  

## <a name="get-more-out-of-your-data-in-kusto-using-machine-learning"></a>Saque más provecho de los datos en Kusto con Machine Learning 

Hay muchos casos de uso interesantes para aprovechar los algoritmos de aprendizaje automático y obtener información interesante de datos de telemetría. Aunque a menudo estos algoritmos requieren un conjunto de datos muy estructurado como entrada, los datos de registro sin procesar no suelen coincidir con el tamaño y la estructura necesarios. 

Nuestro viaje comienza por buscar anomalías en la tasa de errores de un servicio específico de inferencias de Bing. La tabla registros tiene registros 65B y la consulta simple siguiente filtra los errores de 250.000 y crea un recuento de errores de serie temporal que emplea la función de detección de anomalías [series_decompose_anomalies](series-decompose-anomaliesfunction.md). El servicio Kusto detecta las anomalías y se resaltan como puntos rojos en el gráfico de serie temporal.

```kusto
Logs
| where Timestamp >= datetime(2015-08-22) and Timestamp < datetime(2015-08-23) 
| where Level == "e" and Service == "Inferences.UnusualEvents_Main" 
| summarize count() by bin(Timestamp, 5min)
| render anomalychart 
```

El servicio identificó pocos depósitos de tiempo con una tasa de errores sospechosa. Estoy usando Kusto para acercar este período de tiempo y ejecutar una consulta que se agrega en la columna ' mensaje ' intentando buscar los errores principales. He recortado las partes pertinentes del seguimiento completo de la pila del mensaje para ajustarse mejor a la página. Puede ver que tenía un buen éxito con los ocho errores principales, pero, a continuación, alcanzó una cola de errores larga, ya que el mensaje de error se creó con una cadena de formato que contenía datos modificados. 

```kusto
Logs
| where Timestamp >= datetime(2015-08-22 05:00) and Timestamp < datetime(2015-08-22 06:00)
| where Level == "e" and Service == "Inferences.UnusualEvents_Main"
| summarize count() by Message 
| top 10 by count_ 
| project count_, Message 
```

|count_|Message
|---|---
|7125|Error de ExecuteAlgorithmMethod para el método ' RunCycleFromInterimData '...
|7125|Error en la llamada a InferenceHostService.. System. NullReferenceException: referencia a objeto no establecida en una instancia de un objeto...
|7124|Error inesperado del sistema.. System. NullReferenceException: referencia a objeto no establecida en una instancia de un objeto... 
|5112|Error inesperado del sistema.. System. NullReferenceException: referencia a objeto no establecida en una instancia de un objeto.
|174|Error en la llamada a InferenceHostService.. System. ServiceModel. CommunicationException: error al escribir en la canalización:...
|10|Error de ExecuteAlgorithmMethod para el método ' RunCycleFromInterimData '...
|10|Error de inferencia del sistema.. Microsoft. Bing. Platform. inferencias. Service. managers. UserInterimDataManagerException:...
|3|Error en la llamada a InferenceHostService.. System. ServiceModel. CommunicationObjectFaultedException:...
|1|Error de inferencia del sistema... SocialGraph. jefe. OperationResponse... AIS TraceId: 8292FC561AC64BED8FA243808FE74EFD...
|1|Error de inferencia del sistema... SocialGraph. jefe. OperationResponse... AIS TraceId: 5F79F7587FF943EC9B641E02E701AFBF...

Aquí es donde el `reduce` operador le ayudará. El `reduce` operador identificó 63 errores diferentes, tal y como se originó en el mismo punto de instrumentación de seguimiento en el código, y me ayudó a centrarme en el seguimiento de errores significativos adicional en ese período de tiempo.

```kusto
Logs
| where Timestamp >= datetime(2015-08-22 05:00) and Timestamp < datetime(2015-08-22 06:00)
| where Level == "e" and Service == "Inferences.UnusualEvents_Main"
| reduce by Message with threshold=0.35
| project Count, Pattern
```

|Count|Patrón
|---|---
|7125|Error de ExecuteAlgorithmMethod para el método ' RunCycleFromInterimData '...
|  7125|Error en la llamada a InferenceHostService.. System. NullReferenceException: referencia a objeto no establecida en una instancia de un objeto...
|  7124|Error inesperado del sistema.. System. NullReferenceException: referencia a objeto no establecida en una instancia de un objeto...
|  5112|Error inesperado del sistema.. System. NullReferenceException: referencia a objeto no establecida en una instancia de un objeto.
|  174|Error en la llamada a InferenceHostService.. System. ServiceModel. CommunicationException: error al escribir en la canalización:...
|  63|Error de inferencia del sistema.. Microsoft. Bing. Platform. inferencias. \* : Write \* para escribir en el objeto jefe. \* : SocialGraph. jefe. solicitud...
|  10|Error de ExecuteAlgorithmMethod para el método ' RunCycleFromInterimData '...
|  10|Error de inferencia del sistema.. Microsoft. Bing. Platform. inferencias. Service. managers. UserInterimDataManagerException:...
|  3|Error en la llamada a InferenceHostService.. System. ServiceModel. \* : el objeto, System. ServiceModel. Channels. \* + \* , para \* \* es \* ...   en Syst...

Ahora que tengo una buena vista de los principales errores que han contribuido a las anomalías detectadas, deseo comprender el impacto de estos errores en el sistema. La tabla "registros" contiene datos más dimensionales, como "componente", "clúster", etc. El nuevo complemento ' AutoCluster ' puede ayudarme a derivar esa información con una consulta simple. En este ejemplo, puedo ver claramente que cada uno de los cuatro errores principales es específico de un componente y, mientras que los tres primeros errores son específicos del clúster de DB4, el cuarto se produce en todos los clústeres.

```kusto
Logs
| where Timestamp >= datetime(2015-08-22 05:00) and Timestamp < datetime(2015-08-22 06:00)
| where Level == "e" and Service == "Inferences.UnusualEvents_Main"
| evaluate autocluster()
```

|Count |Porcentaje (%)|Componente|Clúster|Message
|---|---|---|---|---
|7125|26,64|InferenceHostService|DB4|ExecuteAlgorithmMethod para el método....
|7125|26,64|Componente desconocido|DB4|Error en la llamada a InferenceHostService....
|7124|26,64|InferenceAlgorithmExecutor|DB4|Error inesperado del sistema...
|5112|19,11|InferenceAlgorithmExecutor|*|Error inesperado del sistema... 

## <a name="mapping-values-from-one-set-to-another"></a>Asignación de valores de un conjunto a otro

Un caso de uso común es el uso de la asignación estática de valores que pueden ayudar a la adopción de resultados de forma más elaborada.  
Por ejemplo, considere la posibilidad de tener la siguiente tabla. DeviceModel especifica un modelo del dispositivo, que no es una forma muy cómoda de hacer referencia al nombre del dispositivo.  


|DeviceModel |Count 
|---|---
|iPhone5, 1 |32 
|iPhone3, 2 |432 
|iPhone7, 2 |55 
|iPhone5, 2 |66 

  
Una mejor representación puede ser:  

|FriendlyName |Count 
|---|---
|iPhone 5 |32 
|iPhone 4 |432 
|iPhone 6 |55 
|iPhone5 |66 

Los dos métodos siguientes muestran cómo se puede lograr esto.  

### <a name="mapping-using-dynamic-dictionary"></a>Asignación con Diccionario dinámico

El método siguiente muestra cómo se puede lograr la asignación mediante un diccionario dinámico y los descriptores de acceso dinámicos.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
```kusto
// Data set definition
let Source = datatable(DeviceModel:string, Count:long)
[
  'iPhone5,1', 32,
  'iPhone3,2', 432,
  'iPhone7,2', 55,
  'iPhone5,2', 66,
];
// Query start here
let phone_mapping = dynamic(
  {
    "iPhone5,1" : "iPhone 5",
    "iPhone3,2" : "iPhone 4",
    "iPhone7,2" : "iPhone 6",
    "iPhone5,2" : "iPhone5"
  });
Source 
| project FriendlyName = phone_mapping[DeviceModel], Count
```

|FriendlyName|Count|
|---|---|
|iPhone 5|32|
|iPhone 4|432|
|iPhone 6|55|
|iPhone5|66|



### <a name="mapping-using-static-table"></a>Asignación mediante una tabla estática

El método siguiente muestra cómo se puede lograr la asignación mediante una tabla persistente y un operador de combinación.
 
Crear la tabla de asignación (solo una vez):

```kusto
.create table Devices (DeviceModel: string, FriendlyName: string) 

.ingest inline into table Devices 
    ["iPhone5,1","iPhone 5"]["iPhone3,2","iPhone 4"]["iPhone7,2","iPhone 6"]["iPhone5,2","iPhone5"]
```

Contenido de los dispositivos ahora:

|DeviceModel |FriendlyName 
|---|---
|iPhone5, 1 |iPhone 5 
|iPhone3, 2 |iPhone 4 
|iPhone7, 2 |iPhone 6 
|iPhone5, 2 |iPhone5 


El mismo truco para crear el origen de la tabla de pruebas:

```kusto
.create table Source (DeviceModel: string, Count: int)

.ingest inline into table Source ["iPhone5,1",32]["iPhone3,2",432]["iPhone7,2",55]["iPhone5,2",66]
```


Unirse y proyecto:

```kusto
Devices  
| join (Source) on DeviceModel  
| project FriendlyName, Count
```

Resultado: 

|FriendlyName |Count 
|---|---
|iPhone 5 |32 
|iPhone 4 |432 
|iPhone 6 |55 
|iPhone5 |66 


## <a name="creating-and-using-query-time-dimension-tables"></a>Crear y usar tablas de dimensiones en tiempo de consulta

En muchos casos, uno desea combinar los resultados de una consulta con alguna tabla de dimensiones ad hoc que no esté almacenada en la base de datos. Es posible definir una expresión cuyo resultado sea una tabla en el ámbito de una sola consulta, de forma similar a la siguiente:

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
// Create a query-time dimension table using datatable
let DimTable = datatable(EventType:string, Code:string)
  [
    "Heavy Rain", "HR",
    "Tornado",    "T"
  ]
;
DimTable
| join StormEvents on EventType
| summarize count() by Code
```

Este es un ejemplo ligeramente más complejo:

```kusto
// Create a query-time dimension table using datatable
let TeamFoundationJobResult = datatable(Result:int, ResultString:string)
  [
    -1, 'None', 0, 'Succeeded', 1, 'PartiallySucceeded', 2, 'Failed',
    3, 'Stopped', 4, 'Killed', 5, 'Blocked', 6, 'ExtensionNotFound',
    7, 'Inactive', 8, 'Disabled', 9, 'JobInitializationError'
  ]
;
JobHistory
  | where PreciseTimeStamp > ago(1h)
  | where Service  != "AX"
  | where Plugin has "Analytics"
  | sort by PreciseTimeStamp desc
  | join kind=leftouter TeamFoundationJobResult on Result
  | extend ExecutionTimeSpan = totimespan(ExecutionTime)
  | project JobName, StartTime, ExecutionTimeSpan, ResultString, ResultMessage
```

## <a name="retrieving-the-latest-by-timestamp-records-per-identity"></a>Recuperación de los registros más recientes (por marca de tiempo) por identidad

Supongamos que tiene una tabla que incluye una `id` columna (que identifica la entidad a la que está asociada cada fila, como un identificador de usuario o un identificador de nodo) y una `timestamp` columna (que proporciona la referencia de tiempo para la fila), así como otras columnas. El objetivo es escribir una consulta que devuelva los 2 registros más recientes para cada valor de la `id` columna, donde "latest" se define como "tener el valor más alto de `timestamp` ".

Esto puede hacerse mediante el [operador de nivel superior anidado](topnestedoperator.md).
En primer lugar, proporcionamos la consulta y, a continuación, la explicaremos:

```kusto
datatable(id:string, timestamp:datetime, bla:string)           // (1)
  [
  "Barak",  datetime(2015-01-01), "1",
  "Barak",  datetime(2016-01-01), "2",
  "Barak",  datetime(2017-01-20), "3",
  "Donald", datetime(2017-01-20), "4",
  "Donald", datetime(2017-01-18), "5",
  "Donald", datetime(2017-01-19), "6"
  ]
| top-nested   of id        by dummy0=max(1),                  // (2)
  top-nested 2 of timestamp by dummy1=max(timestamp),          // (3)
  top-nested   of bla       by dummy2=max(1)                   // (4)
| project-away dummy0, dummy1, dummy2                          // (5)
```

Notas
1. `datatable`Es simplemente una manera de generar algunos datos de prueba para fines de demostración. En realidad, por supuesto, debería tener los datos aquí.
2. Esencialmente, esta línea significa "devolver todos los valores distintos de `id` ".
3. Después, esta línea devuelve, para los 2 registros superiores que maximizan la `timestamp` columna, las columnas del nivel anterior (aquí, solo `id` ) y la columna especificada en este nivel (aquí, `timestamp` ).
4. Esta línea agrega los valores de la `bla` columna para cada uno de los registros devueltos por el nivel anterior. Si la tabla tiene otras columnas de interés, una repetiría esta línea para cada columna de ese tipo.
5. Por último, usamos el [operador de proyección](projectawayoperator.md) para quitar las columnas "adicionales" introducidas por `top-nested` .

## <a name="extending-a-table-with-some-percent-of-total-calculation"></a>Extender una tabla con un cálculo de porcentaje del total

A menudo, cuando una tiene una expresión tabular que incluye una columna numérica, es deseable presentar esa columna al usuario junto con su valor como un porcentaje del total. Por ejemplo, suponga que hay alguna consulta cuyo valor es la tabla siguiente:

|SomeSeries|SomeInt|
|----------|-------|
|Foo       |    100|
|Barra       |    200|

Y desea mostrar esta tabla como:

|SomeSeries|SomeInt|Porcentaje |
|----------|-------|----|
|Foo       |    100|33,3|
|Barra       |    200|66,6|

Para ello, hay que calcular el total (SUM) de la `SomeInt` columna y, a continuación, dividir cada valor de esta columna por el total. Es posible hacerlo para resultados arbitrarios al asignar a estos resultados un nombre mediante el [operador as](asoperator.md):

```kusto
// The following table literal represents a long calculation
// that ends up with an anonymous tabular value:
datatable (SomeInt:int, SomeSeries:string) [
  100, "Foo",
  200, "Bar",
]
// We now give this calculation a name ("X"):
| as X
// Having this name we can refer to it in the sub-expression
// "X | summarize sum(SomeInt)":
| extend Pct = 100 * bin(todouble(SomeInt) / toscalar(X | summarize sum(SomeInt)), 0.001)
```

## <a name="performing-aggregations-over-a-sliding-window"></a>Realizar agregaciones en una ventana deslizante
En el ejemplo siguiente se muestra cómo resumir columnas mediante una ventana deslizante. Permite tomar, por ejemplo, la tabla siguiente, que contiene los precios de los frutos por marcas de tiempo. Supongamos que desea calcular el costo mínimo, máximo y de suma de cada fruta por día, con una ventana deslizante de 7 días. En otras palabras, cada registro del conjunto de resultados agrega los últimos 7 días y el resultado contiene un registro por día en el período de análisis.  

Tabla Fruits: 

|Timestamp|Frutas|Price|
|---|---|---|
|2018-09-24 21:00:00.0000000|Plátanos|3|
|2018-09-25 20:00:00.0000000|Apples (Manzanas)|9|
|2018-09-26 03:00:00.0000000|Plátanos|4|
|2018-09-27 10:00:00.0000000|Ciruelas|8|
|2018-09-28 07:00:00.0000000|Plátanos|6|
|2018-09-29 21:00:00.0000000|Plátanos|8|
|2018-09-30 01:00:00.0000000|Ciruelas|2|
|2018-10-01 05:00:00.0000000|Plátanos|0|
|2018-10-02 02:00:00.0000000|Plátanos|0|
|2018-10-03 13:00:00.0000000|Ciruelas|4|
|2018-10-04 14:00:00.0000000|Apples (Manzanas)|8|
|2018-10-05 05:00:00.0000000|Plátanos|2|
|2018-10-06 08:00:00.0000000|Ciruelas|8|
|2018-10-07 12:00:00.0000000|Plátanos|0|

Consulta de agregación de ventana deslizante (la explicación se proporciona a continuación de los resultados de la consulta): 

```kusto
let _start = datetime(2018-09-24);
let _end = _start + 13d; 
Fruits 
| extend _bin = bin_at(Timestamp, 1d, _start) // #1 
| extend _endRange = iif(_bin + 7d > _end, _end, 
                            iff( _bin + 7d - 1d < _start, _start,
                                iff( _bin + 7d - 1d < _bin, _bin,  _bin + 7d - 1d)))  // #2
| extend _range = range(_bin, _endRange, 1d) // #3
| mv-expand _range to typeof(datetime) limit 1000000 // #4
| summarize min(Price), max(Price), sum(Price) by Timestamp=bin_at(_range, 1d, _start) ,  Fruit // #5
| where Timestamp >= _start + 7d; // #6

```

|Timestamp|Frutas|min_Price|max_Price|sum_Price|
|---|---|---|---|---|
|2018-10-01 00:00:00.0000000|Apples (Manzanas)|9|9|9|
|2018-10-01 00:00:00.0000000|Plátanos|0|8|18|
|2018-10-01 00:00:00.0000000|Ciruelas|2|8|10|
|2018-10-02 00:00:00.0000000|Plátanos|0|8|18|
|2018-10-02 00:00:00.0000000|Ciruelas|2|8|10|
|2018-10-03 00:00:00.0000000|Ciruelas|2|8|14|
|2018-10-03 00:00:00.0000000|Plátanos|0|8|14|
|2018-10-04 00:00:00.0000000|Plátanos|0|8|14|
|2018-10-04 00:00:00.0000000|Ciruelas|2|4|6|
|2018-10-04 00:00:00.0000000|Apples (Manzanas)|8|8|8|
|2018-10-05 00:00:00.0000000|Plátanos|0|8|10|
|2018-10-05 00:00:00.0000000|Ciruelas|2|4|6|
|2018-10-05 00:00:00.0000000|Apples (Manzanas)|8|8|8|
|2018-10-06 00:00:00.0000000|Ciruelas|2|8|14|
|2018-10-06 00:00:00.0000000|Plátanos|0|2|2|
|2018-10-06 00:00:00.0000000|Apples (Manzanas)|8|8|8|
|2018-10-07 00:00:00.0000000|Plátanos|0|2|2|
|2018-10-07 00:00:00.0000000|Ciruelas|4|8|12|
|2018-10-07 00:00:00.0000000|Apples (Manzanas)|8|8|8|

Detalles de la consulta: 

La consulta "estira" (duplica) cada registro de la tabla de entrada en los siete días posteriores a su apariencia real, de modo que cada registro aparece realmente siete veces. Como resultado, al realizar la agregación cada día, la agregación incluye todos los registros de los 7 días anteriores.

Explicación paso a paso (los números hacen referencia a los números de comentarios en línea de consulta):
1. Bin cada registro en 1D (con respecto a _start). 
2. Determine el final del intervalo por registro-_bin + 7D, a menos que esté fuera del intervalo _(inicial, final)_ , en cuyo caso se ajusta. 
3. Para cada registro, cree una matriz de 7 días (marcas de tiempo), empezando por el día del registro actual. 
4. MV: expanda la matriz y, de este modo, duplique cada registro en 7 registros, 1 día separado entre sí. 
5. Realice la función de agregación para cada día. Debido a #4, en realidad se resumen los _últimos_ 7 días. 
6. Por último, dado que los datos del primer 7D están incompletos (no hay ningún período 7D lookback para los primeros 7 días), se excluyen los siete primeros días del resultado final (solo participan en la agregación para 2018-10-01). 

## <a name="find-preceding-event"></a>Buscar evento anterior
En el ejemplo siguiente se muestra cómo buscar un evento anterior entre dos conjuntos de datos.  

*Propósito:* : dados 2 conjuntos de datos, a y B, para cada registro de B buscar su evento anterior en una (es decir, el registro de Arg_max en un que todavía es "anterior" a B). Por ejemplo, a continuación se muestra la salida esperada para los siguientes conjuntos de datos de ejemplo: 

```kusto
let A = datatable(Timestamp:datetime, Id:string, EventA:string)
[
    datetime(2019-01-01 00:00:00), "x", "Ax1",
    datetime(2019-01-01 00:00:01), "x", "Ax2",
    datetime(2019-01-01 00:00:02), "y", "Ay1",
    datetime(2019-01-01 00:00:05), "y", "Ay2",
    datetime(2019-01-01 00:00:00), "z", "Az1"
];
let B = datatable(Timestamp:datetime, Id:string, EventB:string)
[
    datetime(2019-01-01 00:00:03), "x", "B",
    datetime(2019-01-01 00:00:04), "x", "B",
    datetime(2019-01-01 00:00:04), "y", "B",
    datetime(2019-01-01 00:02:00), "z", "B"
];
A; B
```

|Timestamp|Identificador|EventB|
|---|---|---|
|2019-01-01 00:00:00.0000000|x|Ax1|
|2019-01-01 00:00:00.0000000|z|Az1|
|2019-01-01 00:00:01.0000000|x|Ax2|
|2019-01-01 00:00:02.0000000|y|Ay1|
|2019-01-01 00:00:05.0000000|y|Ay2|

</br>

|Timestamp|Identificador|Eventoa|
|---|---|---|
|2019-01-01 00:00:03.0000000|x|B|
|2019-01-01 00:00:04.0000000|x|B|
|2019-01-01 00:00:04.0000000|y|B|
|2019-01-01 00:02:00.0000000|z|B|

Resultado esperado: 

|Identificador|Timestamp|EventB|A_Timestamp|Eventoa|
|---|---|---|---|---|
|x|2019-01-01 00:00:03.0000000|B|2019-01-01 00:00:01.0000000|Ax2|
|x|2019-01-01 00:00:04.0000000|B|2019-01-01 00:00:01.0000000|Ax2|
|y|2019-01-01 00:00:04.0000000|B|2019-01-01 00:00:02.0000000|Ay1|
|z|2019-01-01 00:02:00.0000000|B|2019-01-01 00:00:00.0000000|Az1|

Se sugieren dos enfoques diferentes para este problema. Debe probar ambos en su conjunto de datos específico para encontrar el más adecuado para usted (pueden ejecutarse de manera diferente en conjuntos de datos diferentes). 

### <a name="suggestion-1"></a>#1 de sugerencias:
En esta sugerencia se serializan ambos conjuntos de datos por identificador y marca de tiempo; a continuación, se agrupan todos los eventos B con todos los eventos A anteriores y se elige el `arg_max` fuera de todos como en el grupo. 

```kusto
A
| extend A_Timestamp = Timestamp, Kind="A"
| union (B | extend B_Timestamp = Timestamp, Kind="B")
| order by Id, Timestamp asc 
| extend t = iff(Kind == "A" and (prev(Kind) != "A" or prev(Id) != Id), 1, 0)
| extend t = row_cumsum(t)
| summarize Timestamp=make_list(Timestamp), EventB=make_list(EventB), arg_max(A_Timestamp, EventA) by t, Id
| mv-expand Timestamp to typeof(datetime), EventB to typeof(string)
| where isnotempty(EventB)
| project-away t
```

### <a name="suggestion-2"></a>#2 de sugerencias:
Esta sugerencia requiere la definición de un período de lookback máximo (¿cuánto "más antiguo" se permite que el registro de un se compare con B?) y, a continuación, combina los dos conjuntos de datos en el identificador y este período de lookback. La combinación produce todos los posibles candidatos (todos los registros A que son anteriores a B y dentro del período de lookback) y, a continuación, filtramos la más cercana a B de arg_min (TimestampB – Timestampa). Cuanto menor sea el período de lookback, se espera que la consulta funcione mejor. 

En el ejemplo siguiente, el período lookback se establece en 1m y, por lo tanto, el registro con el identificador ' z ' no tiene un evento ' A ' correspondiente (ya que es un valor más antiguo de 2 m).

```kusto
let _maxLookbackPeriod = 1m;  
let _internalWindowBin = _maxLookbackPeriod / 2;
let B_events = B 
    | extend ID = new_guid()
    | extend _time = bin(Timestamp, _internalWindowBin)
    | extend _range = range(_time - _internalWindowBin, _time + _maxLookbackPeriod, _internalWindowBin) 
    | mv-expand _range to typeof(datetime) 
    | extend B_Timestamp = Timestamp, _range;
let A_events = A 
    | extend _time = bin(Timestamp, _internalWindowBin)
    | extend _range = range(_time - _internalWindowBin, _time + _maxLookbackPeriod, _internalWindowBin) 
    | mv-expand _range to typeof(datetime) 
    | extend A_Timestamp = Timestamp, _range;
B_events
    | join kind=leftouter (
        A_events
) on Id, _range
| where isnull(A_Timestamp) or (A_Timestamp <= B_Timestamp and B_Timestamp <= A_Timestamp + _maxLookbackPeriod)
| extend diff = coalesce(B_Timestamp - A_Timestamp, _maxLookbackPeriod*2)
| summarize arg_min(diff, *) by ID
| project Id, B_Timestamp, A_Timestamp, EventB, EventA
```

|Identificador|B_Timestamp|A_Timestamp|EventB|Eventoa|
|---|---|---|---|---|
|x|2019-01-01 00:00:03.0000000|2019-01-01 00:00:01.0000000|B|Ax2|
|x|2019-01-01 00:00:04.0000000|2019-01-01 00:00:01.0000000|B|Ax2|
|y|2019-01-01 00:00:04.0000000|2019-01-01 00:00:02.0000000|B|Ay1|
|z|2019-01-01 00:02:00.0000000||B||
