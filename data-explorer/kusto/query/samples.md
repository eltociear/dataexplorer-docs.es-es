---
title: 'Ejemplos: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen ejemplos en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 746017d41b5f1a13ce73f2c27df9cac5b5982ff0
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663591"
---
# <a name="samples"></a>Ejemplos

A continuación se muestran algunas necesidades de consulta comunes y cómo se puede usar el lenguaje de consulta Kusto para satisfacerlas.

## <a name="display-a-column-chart"></a>Mostrar un gráfico de columnas

Proyecte dos o más columnas y utilícelas como los ejes x e y de un gráfico:

```kusto 
StormEvents
| where isnotempty(EndLocation) 
| summarize event_count=count() by EndLocation
| top 10 by event_count
| render columnchart
```

* La primera columna forma el eje X. Puede ser numérico, datetime o string. 
* Utilice `where` `summarize` , `top` y para limitar el volumen de datos que muestra.
* Ordene los resultados para definir el orden del eje X.

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

Se [`let`](./letstatement.md) utiliza para nombrar una proyección de la tabla que se reduce en la medida de lo posible antes de entrar en la unión.
[`Project`](./projectoperator.md)se utiliza para cambiar los nombres de las marcas de tiempo para que las horas de inicio y detención puedan aparecer en el resultado. También selecciona las otras columnas que se quieren ver en el resultado. [`join`](./joinoperator.md)coincide con las entradas de inicio y detención para la misma actividad, creando una fila para cada actividad. Por último, `project` agrega de nuevo una columna para mostrar la duración de la actividad.


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

A continuación, agrupamos por hora de inicio e IP para obtener un grupo para cada sesión. Debemos proporcionar `bin` una función para el parámetro StartTime: si no lo hacemos, Kusto usará automáticamente bins de 1 hora, que coincidirán con algunas horas de inicio con los tiempos de parada incorrectos.

`arg_min`selecciona la fila con la duración más `*` pequeña en cada grupo y el parámetro pasa a través de todas las demás columnas, aunque prefija "min_" a cada nombre de columna. 

:::image type="content" source="images/samples/040.png" alt-text="040"::: 

A continuación, podemos agregar código para contar las duraciones en ubicaciones de tamaño conveniente. Tenemos una ligera preferencia por un gráfico de `1s` barras, así que dividimos por convertir los intervalos de tiempo en números. 


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

Pero en lugar de mantener esos arrays, los ampliaremos usando [mv-expand](./mvexpandoperator.md):

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

* Necesitamos todatetime() porque [mv-expand](./mvexpandoperator.md) produce una columna de tipo dinámico.
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

## <a name="introduce-null-bins-into-summarize"></a>Introducir bins nulos en resumen

Cuando `summarize` el operador se aplica sobre una `datetime` clave de grupo que consta de una columna, normalmente "bins" esos valores a bins de ancho fijo. Por ejemplo:

```kusto
let StartTime=ago(12h);
let StopTime=now()
T
| where Timestamp > StartTime and Timestamp <= StopTime 
| where ...
| summarize Count=count() by bin(Timestamp, 5m)
```

Esta operación genera una tabla con una `T` sola fila por grupo de filas que se encuentran en cada bin de cinco minutos. Lo que no hace es agregar "bins nulos": `StartTime` `StopTime` filas para los valores de `T`ubicación de tiempo entre y para las que no hay ninguna fila correspondiente en . 

A menudo, se desea "pad" la mesa con esas bandejas. Aquí hay una manera de hacerlo:

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

Aquí está una explicación paso a paso de la consulta anterior: 

1. El `union` uso del operador nos permite agregar filas adicionales a una tabla. Esas filas las produce la `union`expresión a .
2. Usar `range` el operador para producir una tabla que tenga una sola fila/columna.
   La tabla no se utiliza `mv-expand` para nada más que para trabajar.
3. Uso `mv-expand` del operador `range` sobre la función para crear tantas filas `StartTime` `EndTime`como haya bins de 5 minutos entre y .
4. Todo con `Count` `0`un de .
5. Por último, `summarize` usamos el operador para agrupar ubicaciones desde el `union` argumento original (izquierda o externa) y bins desde el argumento interno a él (es decir, las filas bin nulas). Esto garantiza que la salida tiene una fila por ubicación, cuyo valor es cero o el recuento original.  

## <a name="get-more-out-of-your-data-in-kusto-using-machine-learning"></a>Obtenga más de sus datos en Kusto con Machine Learning 

Hay muchos casos de uso interesantes para aprovechar los algoritmos de aprendizaje automático y obtener información interesante de los datos de telemetría. Aunque a menudo estos algoritmos requieren un conjunto de datos muy estructurado como su entrada, los datos de registro sin procesar normalmente no coincidirán con la estructura y el tamaño requeridos. 

Nuestro viaje comienza con la búsqueda de anomalías en la tasa de error de un servicio específico de Inferencias de Bing. La tabla Logs tiene registros 65B y la consulta simple debajo filtra los errores 250K y crea una serie temporal de datos de recuento de errores que utiliza la función de detección de anomalías [series_decompose_anomalies](series-decompose-anomaliesfunction.md). Las anomalías son detectadas por el servicio Kusto, y se resaltan como puntos rojos en el gráfico de series temporales.

```kusto
Logs
| where Timestamp >= datetime(2015-08-22) and Timestamp < datetime(2015-08-23) 
| where Level == "e" and Service == "Inferences.UnusualEvents_Main" 
| summarize count() by bin(Timestamp, 5min)
| render anomalychart 
```

El servicio identificó algunos buckets de tiempo con una tasa de error sospechosa. Estoy usando Kusto para ampliar este período de tiempo, ejecutando una consulta que agrega en la columna 'Mensaje' tratando de buscar los errores principales. He recortado las partes relevantes de toda la pila de seguimiento del mensaje para encajar mejor en la página. Puede ver que tuve un buen éxito con los ocho errores principales, pero luego alcanzó una larga cola de errores ya que el mensaje de error fue creado por una cadena de formato que contenía datos cambiantes. 

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
|7125|ExecuteAlgorithmMethod para el método 'RunCycleFromInterimData' ha fallado...
|7125|Error en la llamada a InferenceHostService. System.NullReferenceException: referencia de objeto no establecida en una instancia de un objeto...
|7124|Error inesperado del sistema de inferencia. System.NullReferenceException: referencia de objeto no establecida en una instancia de un objeto... 
|5112|Error inesperado del sistema de inferencia. System.NullReferenceException: referencia de objeto no establecida en una instancia de un objeto..
|174|InferenceHostService llamada failed..System.ServiceModel.CommunicationException: se ha producido un error al escribir en la canalización:...
|10|ExecuteAlgorithmMethod para el método 'RunCycleFromInterimData' ha fallado...
|10|Error del sistema de inferencia. Microsoft.Bing.Platform.Inferences.Service.Managers.UserInterimDataManagerException:...
|3|Error en la llamada a InferenceHostService..System.ServiceModel.CommunicationObjectFaultedException:...
|1|Error del sistema de inferencia... SocialGraph.BOSS.OperationResponse... AIS TraceId:8292FC561AC64BED8FA243808FE74EFD...
|1|Error del sistema de inferencia... SocialGraph.BOSS.OperationResponse... AIS TraceId: 5F79F7587FF943EC9B641E02E701AFBF...

Aquí es `reduce` donde el operador viene a ayudar. El `reduce` operador identificó 63 errores diferentes originados por el mismo punto de instrumentación de seguimiento en el código y me ayudó a centrarme en el seguimiento de errores significativo adicional en esa ventana de tiempo.

```kusto
Logs
| where Timestamp >= datetime(2015-08-22 05:00) and Timestamp < datetime(2015-08-22 06:00)
| where Level == "e" and Service == "Inferences.UnusualEvents_Main"
| reduce by Message with threshold=0.35
| project Count, Pattern
```

|Count|Modelo
|---|---
|7125|ExecuteAlgorithmMethod para el método 'RunCycleFromInterimData' ha fallado...
|  7125|Error en la llamada a InferenceHostService. System.NullReferenceException: referencia de objeto no establecida en una instancia de un objeto...
|  7124|Error inesperado del sistema de inferencia. System.NullReferenceException: referencia de objeto no establecida en una instancia de un objeto...
|  5112|Error inesperado del sistema de inferencia. System.NullReferenceException: referencia de objeto no establecida en una instancia de un objeto..
|  174|InferenceHostService llamada failed..System.ServiceModel.CommunicationException: se ha producido un error al escribir en la canalización:...
|  63|Error del sistema de inferencia. Microsoft.Bing.Platform.Inferences. \*: \* Escriba para escribir en el objeto BOSS. \*: SocialGraph.BOSS.Reques...
|  10|ExecuteAlgorithmMethod para el método 'RunCycleFromInterimData' ha fallado...
|  10|Error del sistema de inferencia. Microsoft.Bing.Platform.Inferences.Service.Managers.UserInterimDataManagerException:...
|  3|Error en la llamada a InferenceHostService. System.ServiceModel. \*: el objeto System.ServiceModel.Channels. \* \* \*, \* porque es el ... + \*   en Syst...

Ahora que tengo una buena vista de los principales errores que contribuyeron a las anomalías detectadas, quiero entender el impacto de estos errores en todo mi sistema. La tabla 'Logs' contiene datos dimensionales adicionales como 'Component', 'Cluster', etc... El nuevo plugin 'autocluster' puede ayudarme a derivar esa información con una simple consulta. En este ejemplo siguiente, puedo ver claramente que cada uno de los cuatro errores principales es específico de un componente, y aunque los tres errores principales son específicos del clúster de DB4, el cuarto ocurre en todos los clústeres.

```kusto
Logs
| where Timestamp >= datetime(2015-08-22 05:00) and Timestamp < datetime(2015-08-22 06:00)
| where Level == "e" and Service == "Inferences.UnusualEvents_Main"
| evaluate autocluster()
```

|Count |Porcentaje (%)|Componente|Clúster|Message
|---|---|---|---|---
|7125|26.64|InferenceHostService|DB4|ExecuteAlgorithmMethod para el método ....
|7125|26.64|Componente desconocido|DB4|Error en la llamada a InferenceHostService....
|7124|26.64|InferenceAlgorithmExecutor|DB4|Error inesperado del sistema de inferencia...
|5112|19.11|InferenceAlgorithmExecutor|*|Error inesperado del sistema de inferencia... 

## <a name="mapping-values-from-one-set-to-another"></a>Asignación de valores de un conjunto a otro

Un caso de uso común es el uso de la asignación estática de valores que pueden ayudar a adoptar los resultados de una manera más presentable.  
Por ejemplo, considere la posibilidad de tener la siguiente tabla. DeviceModel especifica un modelo del dispositivo, que no es una forma muy conveniente de hacer referencia al nombre del dispositivo.  


|DeviceModel |Count 
|---|---
|iPhone5,1 |32 
|iPhone3,2 |432 
|iPhone7,2 |55 
|iPhone5,2 |66 

  
Una mejor representación puede ser:  

|FriendlyName |Count 
|---|---
|iPhone 5 |32 
|iPhone 4 |432 
|iPhone 6 |55 
|iPhone5 |66 

Los dos enfoques siguientes demuestran cómo se puede lograr esto.  

### <a name="mapping-using-dynamic-dictionary"></a>Asignación mediante diccionario dinámico

El enfoque siguiente muestra cómo se puede lograr la asignación mediante un diccionario dinámico y descriptores de acceso dinámicos.

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



### <a name="mapping-using-static-table"></a>Asignación mediante tabla estática

El enfoque siguiente muestra cómo se puede lograr la asignación mediante una tabla persistente y un operador de combinación.
 
Cree la tabla de asignación (solo una vez):

```kusto
.create table Devices (DeviceModel: string, FriendlyName: string) 

.ingest inline into table Devices 
    ["iPhone5,1","iPhone 5"]["iPhone3,2","iPhone 4"]["iPhone7,2","iPhone 6"]["iPhone5,2","iPhone5"]
```

Contenido de dispositivos ahora:

|DeviceModel |FriendlyName 
|---|---
|iPhone5,1 |iPhone 5 
|iPhone3,2 |iPhone 4 
|iPhone7,2 |iPhone 6 
|iPhone5,2 |iPhone5 


El mismo truco para crear la tabla de prueba Origen:

```kusto
.create table Source (DeviceModel: string, Count: int)

.ingest inline into table Source ["iPhone5,1",32]["iPhone3,2",432]["iPhone7,2",55]["iPhone5,2",66]
```


Unirse y proyectar:

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


## <a name="creating-and-using-query-time-dimension-tables"></a>Creación y uso de tablas de dimensiones en tiempo de consulta

En muchos casos, se desea combinar los resultados de una consulta con alguna tabla de dimensiones ad hoc que no se almacena en la base de datos. Es posible definir una expresión cuyo resultado es una tabla con ámbito a una sola consulta haciendo algo como esto:

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

Este es un ejemplo un poco más complejo:

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

## <a name="retrieving-the-latest-by-timestamp-records-per-identity"></a>Recuperar los registros más recientes (por marca de tiempo) por identidad

Supongamos que tiene una `id` tabla que incluye una columna (que identifica la entidad con la que `timestamp` está asociada cada fila, como un identificador de usuario o un identificador de nodo) y una columna (que proporciona la referencia de tiempo para la fila), así como otras columnas. Su objetivo es escribir una consulta que devuelva los `id` 2 registros más recientes para cada valor `timestamp`de la columna, donde "latest" se define como "tener el valor más alto de ".

Esto se puede hacer mediante el [operador anidado superior.](topnestedoperator.md)
Primero proporcionamos la consulta y luego la explicaremos:

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
1. Es `datatable` sólo una manera de producir algunos datos de prueba con fines de demostración. En realidad, por supuesto, tendrías los datos aquí.
2. Esta línea significa esencialmente "devolver `id`todos los valores distintos de ".
3. A continuación, esta línea devuelve, para `timestamp` los 2 registros principales que `id`maximizan la columna, las columnas `timestamp`del nivel anterior (aquí, solo ) y la columna especificada en este nivel (aquí, ).
4. Esta línea agrega los `bla` valores de la columna para cada uno de los registros devueltos por el nivel anterior. Si la tabla tiene otras columnas de interés, se repetiría esta línea para cada columna de este tipo.
5. Por último, usamos el [operador project-away](projectawayoperator.md) para quitar `top-nested`las columnas "extra" introducidas por .

## <a name="extending-a-table-with-some-percent-of-total-calculation"></a>Ampliar una tabla con un cálculo porcentual del total

A menudo, cuando uno tiene una expresión tabular que incluye una columna numérica, es deseable presentar esa columna al usuario junto con su valor como un porcentaje del total. Por ejemplo, supongamos que hay alguna consulta cuyo valor es la tabla siguiente:

|SomeSeries|SomeInt|
|----------|-------|
|Foo       |    100|
|Barra       |    200|

Y desea mostrar esta tabla como:

|SomeSeries|SomeInt|Pct |
|----------|-------|----|
|Foo       |    100|33,3|
|Barra       |    200|66,6|

Para ello, es necesario calcular el total `SomeInt` (suma) de la columna y, a continuación, dividir cada valor de esta columna por el total. Es posible hacerlo para resultados arbitrarios dando a estos resultados un nombre utilizando el [operador as](asoperator.md):

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

## <a name="performing-aggregations-over-a-sliding-window"></a>Realización de agregaciones a través de una ventana deslizante
En el ejemplo siguiente se muestra cómo resumir columnas mediante una ventana deslizante. Tomemos, por ejemplo, la siguiente tabla, que contiene los precios de las frutas por marcas de tiempo. Supongamos que nos gustaría calcular el mínimo, máximo y suma de costo de cada fruta por día, utilizando una ventana deslizante de 7 días. En otras palabras, cada registro del conjunto de resultados agrega los últimos 7 días y el resultado contiene un registro por día en el período de análisis.  

Tabla de frutas: 

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

Consulta de agregación de ventana deslizante (la explicación se proporciona a continuación los resultados de la consulta): 

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

La consulta "estira" (duplica) cada registro de la tabla de entrada a lo largo de 7 días después de su apariencia real, de forma que cada registro realmente aparece 7 veces. Como resultado, al realizar la agregación por cada día, la agregación incluye todos los registros de los 7 días anteriores.

Explicación paso a paso (los números se refieren a los números en los comentarios en línea de la consulta):
1. Bin cada registro en 1d (en relación con _start). 
2. Determine el final del rango por registro - _bin + 7d, a menos que esté fuera del rango _(inicio, fin),_ en cuyo caso se ajusta. 
3. Para cada registro, cree una matriz de 7 días (marcas de tiempo), a partir del día del registro actual. 
4. mv-expand la matriz, duplicando así cada registro a 7 registros, 1 día aparte entre sí. 
5. Realice la función de agregación para cada día. Debido a #4, esto resume los _últimos_ 7 días. 
6. Por último, dado que los datos del primer 7d están incompletos (no hay un período de retención de 7d para los primeros 7 días), excluimos los primeros 7 días del resultado final (solo participan en la agregación para el 2018-10-01). 

## <a name="find-preceding-event"></a>Buscar evento anterior
En el ejemplo siguiente se muestra cómo encontrar un evento anterior entre 2 conjuntos de datos.  

*Finalidad:* : Dados 2 conjuntos de datos, A y B, para cada registro en B encontrar su evento anterior en A (en otras palabras, el registro de arg_max en A que todavía es "más antiguo" que B). Por ejemplo, a continuación se muestra la salida esperada para los siguientes conjuntos de datos de ejemplo: 

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

|Timestamp|Identificador|EventA|
|---|---|---|
|2019-01-01 00:00:03.0000000|x|B|
|2019-01-01 00:00:04.0000000|x|B|
|2019-01-01 00:00:04.0000000|y|B|
|2019-01-01 00:02:00.0000000|z|B|

Resultado esperado: 

|Identificador|Timestamp|EventB|A_Timestamp|EventA|
|---|---|---|---|---|
|x|2019-01-01 00:00:03.0000000|B|2019-01-01 00:00:01.0000000|Ax2|
|x|2019-01-01 00:00:04.0000000|B|2019-01-01 00:00:01.0000000|Ax2|
|y|2019-01-01 00:00:04.0000000|B|2019-01-01 00:00:02.0000000|Ay1|
|z|2019-01-01 00:02:00.0000000|B|2019-01-01 00:00:00.0000000|Az1|

Hay 2 enfoques diferentes sugeridos para este problema. Debe probar ambos en su conjunto de datos específico para encontrar el más adecuado para usted (pueden funcionar de manera diferente en diferentes conjuntos de datos). 

### <a name="suggestion-1"></a>#1 de sugerencias:
Esta sugerencia serializa ambos conjuntos de datos por identificador y marca de tiempo, `arg_max` a continuación, agrupa todos los eventos B con todos sus eventos A anteriores y selecciona el de todos como en el grupo. 

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

### <a name="suggestion-2"></a>Sugerencia #2:
Esta sugerencia requiere definir un período de retroceso máximo (¿cuánto "más antiguo" permitimos que el registro en A se compare con B?) y, a continuación, se une a los 2 conjuntos de datos en Id y este período de retención. La combinación produce todos los candidatos posibles (todos los registros A que son más antiguos que B y dentro del período de búsqueda), y luego filtramos el más cercano a B por arg_min(TimestampB – TimestampA). Cuanto menor sea el período de búsqueda, se espera que la consulta funcione mejor. 

En el ejemplo siguiente, el período de búsqueda se establece en 1m y, por lo tanto, el registro con Id 'z' no tiene un evento 'A' correspondiente (ya que es 'A' es mayor por 2m).

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

|Identificador|B_Timestamp|A_Timestamp|EventB|EventA|
|---|---|---|---|---|
|x|2019-01-01 00:00:03.0000000|2019-01-01 00:00:01.0000000|B|Ax2|
|x|2019-01-01 00:00:04.0000000|2019-01-01 00:00:01.0000000|B|Ax2|
|y|2019-01-01 00:00:04.0000000|2019-01-01 00:00:02.0000000|B|Ay1|
|z|2019-01-01 00:02:00.0000000||B||