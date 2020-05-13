---
title: 'Unirse dentro de la ventana de tiempo: Azure Explorador de datos'
description: En este artículo se describe la Unión dentro de la ventana de tiempo de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4741da4367bb1a350c7310ea21ebe5ce9b91b06b
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271492"
---
# <a name="joining-within-time-window"></a>Unirse dentro de la ventana de tiempo

A menudo resulta útil unirse entre dos conjuntos de datos de gran tamaño en alguna clave de cardinalidad alta (como un identificador de operación o un identificador de sesión) y limitar aún más los registros de la derecha ( `$right` ) que deben coincidir para cada registro de la izquierda ( `$left` ) agregando una restricción en la "distancia de tiempo" entre `datetime` las columnas de la izquierda y la derecha. Esto difiere de la operación de Unión Kusto habitual como, además de la parte de "combinación de igualdad" (que coincide con la clave de cardinalidad alta o los conjuntos de datos izquierdo y derecho), el sistema también puede aplicar una función de distancia y utilizarla para acelerar considerablemente la combinación. Tenga en cuenta que una función de distancia no se comporta como la igualdad (es decir, cuando tanto `dist(x,y)` como `dist(y,z)` son true, no sigue eso `dist(x,z)` también es cierto). *Internamente, a veces hacemos referencia a esto como "combinación diagonal".*

Por ejemplo, supongamos que deseamos identificar las secuencias de eventos en un período de tiempo relativamente pequeño. Para mostrar este ejemplo, supongamos que tenemos una tabla `T` con el siguiente esquema:

- `SessionId`: Una columna de tipo `string` con identificadores de correlación.
- `EventType`: Una columna de tipo `string` que identifica el tipo de evento del registro.
- `Timestamp`: Una columna de tipo `datetime` que indica cuándo se produjo el evento descrito por el registro.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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

Queremos que la consulta responda a la siguiente pregunta:

   Busque todos los identificadores de sesión en los que el tipo de evento iba `A` seguido de un tipo `B` de evento dentro de la `1min` ventana de tiempo.

(En los datos de ejemplo anteriores, el único identificador de sesión es `0` ).

Semánticamente, la siguiente consulta responde a esta pregunta, aunque es poco eficaz:

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

Para optimizar esta consulta, podemos volver a escribirla tal y como se describe a continuación para que la ventana de tiempo se exprese como una clave de combinación.

**Volver a escribir la consulta para la ventana de tiempo**

La idea es volver a escribir la consulta para que los `datetime` valores se "agrupen" en cubos cuyo tamaño sea la mitad del tamaño de la ventana de tiempo.
A continuación, podemos usar la combinación de igualdad de Kusto para comparar los ID. de cubo.

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

**Referencia de consulta ejecutable (con tabla insertada)**

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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


**consulta de datos 50 millones**

La siguiente consulta emula el conjunto de datos de registros 50 millones y ~ 10 millones de identificadores y ejecuta la consulta con la técnica descrita anteriormente.

<!-- csl: https://help.kusto.windows.net:443/Samples -->
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
