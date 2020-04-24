---
title: Unirse dentro de la ventana de tiempo - Explorador de Azure Data Explorer Microsoft Docs
description: En este artículo se describe la unión dentro de la ventana de tiempo en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: aa3b81694714ef5af94407cdfdac263af0631e40
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81513369"
---
# <a name="joining-within-time-window"></a>Unirse dentro de la ventana de tiempo

A menudo resulta útil unir entre dos grandes conjuntos de datos en alguna clave de alta cardinalidad (como un`$right`identificador de operación o un identificador de`$left`sesión) y limitar aún más los `datetime` registros del lado derecho ( ) que deben coincidir para cada registro del lado izquierdo ( ) agregando una restricción en la "distancia de tiempo" entre las columnas a la izquierda y a la derecha. Esto difiere de la operación de unión Kusto habitual ya que, además de la parte "equi-join" (que coincide con la clave de alta cardinalidad o los conjuntos de datos izquierdo y derecho), el sistema también puede aplicar una función de distancia y utilizarla para acelerar considerablemente la unión. Tenga en cuenta que una función de distancia no `dist(x,y)` `dist(y,z)` se comporta como la `dist(x,z)` igualdad (es decir, cuando ambos y son verdaderos no sigue que también es true.) *Internamente, a veces nos referimos a esto como "unión diagonal".*

Por ejemplo, supongamos que queremos identificar secuencias de eventos dentro de una ventana de tiempo relativamente pequeña. Para demostrar este ejemplo, supongamos que tenemos una tabla `T` con el siguiente esquema:

- `SessionId`: una columna `string` de tipo con identificadores de correlación.
- `EventType`: columna de `string` tipo que identifica el tipo de evento del registro.
- `Timestamp`: una columna `datetime` de tipo que indica cuándo ocurrió el evento descrito por el registro.

```kusto
let T = datatable(SessionId:string, EventType:string, Timestamp:datetime)
[
    '0', 'A', datetime(2017-10-01 00:00:00),
    '0', 'B', datetime(2017-10-01 00:01:00),
    '1', 'B', datetime(2017-10-01 00:02:00),
    '1', 'A', datetime(2017-10-01 00:03:00),
    '3', 'A', datetime(2017-10-01 00:04:00),
    '3', 'B', datetime(2017-10-01 00:10:00),
];
T
```

|SessionId|EventType|Timestamp|
|---|---|---|
|0|Un|2017-10-01 00:00:00.0000000|
|0|B|2017-10-01 00:01:00.0000000|
|1|B|2017-10-01 00:02:00.0000000|
|1|Un|2017-10-01 00:03:00.0000000|
|3|Un|2017-10-01 00:04:00.0000000|
|3|B|2017-10-01 00:10:00.0000000|


**Declaración del problema**

Queremos que nuestra consulta responda a la siguiente pregunta:

   Busque todos los identificadores de `A` sesión en los `B` `1min` que el tipo de evento fue seguido de un tipo de evento dentro de la ventana de tiempo.

(En los datos de ejemplo anteriores, el único ID de sesión es `0`.)

Semánticamente, la siguiente consulta responde a esta pregunta, aunque de manera ineficiente:

```kusto
T 
| where EventType == 'A'
| project SessionId, Start=Timestamp
| join kind=inner
    (
    T 
    | where EventType == 'B'
    | project SessionId, End=Timestamp
    ) on SessionId
| where (End - Start) between (0min .. 1min)
| project SessionId, Start, End 

```

|SessionId|Start|End|
|---|---|---|
|0|2017-10-01 00:00:00.0000000|2017-10-01 00:01:00.0000000|

Para optimizar esta consulta, podemos volver a escribirla como se describe a continuación para que la ventana de tiempo se exprese como una clave de combinación.

**Reescribir la consulta para tener en cuenta la ventana de tiempo**

La idea es reescribir la `datetime` consulta para que los valores se "discretizan" en buckets cuyo tamaño es la mitad del tamaño de la ventana de tiempo.
A continuación, podemos usar la unión equi de Kusto para comparar esos datos de identificación.

```kusto
let lookupWindow = 1min;
let lookupBin = lookupWindow / 2.0; // lookup bin = equal to 1/2 of the lookup window
T 
| where EventType == 'A'
| project SessionId, Start=Timestamp,
          // TimeKey on the left side of the join is mapped to a discrete time axis for the join purpose
          TimeKey = bin(Timestamp, lookupBin)
| join kind=inner
    (
    T 
    | where EventType == 'B'
    | project SessionId, End=Timestamp,
              // TimeKey on the right side of the join - emulates event 'B' appearing several times
              // as if it was 'replicated'
              TimeKey = range(bin(Timestamp-lookupWindow, lookupBin),
                              bin(Timestamp, lookupBin),
                              lookupBin)
    // 'mv-expand' translates the TimeKey array range into a column
    | mv-expand TimeKey to typeof(datetime)
    ) on SessionId, TimeKey 
| where (End - Start) between (0min .. lookupWindow)
| project SessionId, Start, End 
```

|SessionId|Start|End|
|---|---|---|
|0|2017-10-01 00:00:00.0000000|2017-10-01 00:01:00.0000000|


**Referencia de consulta ejecutándose (con tabla insertada)**

```kusto
let T = datatable(SessionId:string, EventType:string, Timestamp:datetime)
[
    '0', 'A', datetime(2017-10-01 00:00:00),
    '0', 'B', datetime(2017-10-01 00:01:00),
    '1', 'B', datetime(2017-10-01 00:02:00),
    '1', 'A', datetime(2017-10-01 00:03:00),
    '3', 'A', datetime(2017-10-01 00:04:00),
    '3', 'B', datetime(2017-10-01 00:10:00),
];
let lookupWindow = 1min;
let lookupBin = lookupWindow / 2.0;
T 
| where EventType == 'A'
| project SessionId, Start=Timestamp, TimeKey = bin(Timestamp, lookupBin)
| join kind=inner
    (
    T 
    | where EventType == 'B'
    | project SessionId, End=Timestamp,
              TimeKey = range(bin(Timestamp-lookupWindow, lookupBin),
                              bin(Timestamp, lookupBin),
                              lookupBin)
    | mv-expand TimeKey to typeof(datetime)
    ) on SessionId, TimeKey 
| where (End - Start) between (0min .. lookupWindow)
| project SessionId, Start, End 
```

|SessionId|Start|End|
|---|---|---|
|0|2017-10-01 00:00:00.0000000|2017-10-01 00:01:00.0000000|


**Consulta de datos de 50M**

La siguiente consulta emula el conjunto de datos de los registros 50M y los identificadores de 10M y ejecuta la consulta con la técnica descrita anteriormente.

```kusto
let T = range x from 1 to 50000000 step 1
| extend SessionId = rand(10000000), EventType = rand(3), Time=datetime(2017-01-01)+(x * 10ms)
| extend EventType = case(EventType <= 1, "A",
                          EventType <= 2, "B",
                          "C");
let lookupWindow = 1min;
let lookupBin = lookupWindow / 2.0;
T 
| where EventType == 'A'
| project SessionId, Start=Time, TimeKey = bin(Time, lookupBin)
| join kind=inner
    (
    T 
    | where EventType == 'B'
    | project SessionId, End=Time, 
              TimeKey = range(bin(Time-lookupWindow, lookupBin), 
                              bin(Time, lookupBin),
                              lookupBin)
    | mv-expand TimeKey to typeof(datetime)
    ) on SessionId, TimeKey 
| where (End - Start) between (0min .. lookupWindow)
| project SessionId, Start, End 
| count 
```

|Count|
|---|
|1276|