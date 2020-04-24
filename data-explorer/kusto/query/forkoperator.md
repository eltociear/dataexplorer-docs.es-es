---
title: 'Operador de horquillas: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador de bifurcación en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 13cd837b3874a55ec758991f5609e089daba7c75
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81515205"
---
# <a name="fork-operator"></a>Operador fork

Ejecuta varios operadores de consumidor en paralelo.

**Sintaxis**

*T* `|` `=``(``)` `=``(`*name*`)` *name**subquery* *subquery* [ nombre ] subconsulta [ nombre ] subconsulta ... `fork`

**Argumentos**

* *subconsulta* es una canalización descendente de operadores de consulta
* *nombre* es un nombre temporal para la tabla de resultados de la subconsulta

**Devuelve**

Varias tablas de resultados, una para cada una de las subconsultas.

**Operadores soportados**

[`as`](asoperator.md), [`count`](countoperator.md), [`extend`](extendoperator.md), [`parse`](parseoperator.md), [`where`](whereoperator.md), [`take`](takeoperator.md), [`project`](projectoperator.md), [`project-away`](projectawayoperator.md), [`summarize`](summarizeoperator.md), [`top`](topoperator.md), [`top-nested`](topnestedoperator.md), [`sort`](sortoperator.md), [`mv-expand`](mvexpandoperator.md), [`reduce`](reduceoperator.md)

**Notas**

* [`materialize`](materializefunction.md)función se puede utilizar [`join`](joinoperator.md) como [`union`](unionoperator.md) un reemplazo para el uso o en las patas de la horquilla.
El flujo de entrada se almacenará en caché mediante materialize y, a continuación, la expresión almacenada en caché se puede utilizar en tramos de unión/unión.

* Un nombre, dado `name` por el [`as`](asoperator.md) argumento o mediante el uso de [`Kusto.Explorer`](../tools/kusto-explorer.md) operador se utilizará como para nombrar la pestaña de resultado en la herramienta.

* Evite `fork` usar con una sola subconsulta.

* Prefiere [batch](batches.md) usar [`materialize`](materializefunction.md) batch con instrucciones `fork` de expresión tabular sobre el operador.

**Ejemplos**

```kusto
KustoLogs
| where Timestamp > ago(1h)
| fork
    ( where Level == "Error" | project EventText | limit 100 )
    ( project Timestamp, EventText | top 1000 by Timestamp desc)
    ( summarize min(Timestamp), max(Timestamp) by ActivityID )
 
// In the following examples the result tables will be named: Errors, EventsTexts and TimeRangePerActivityID
KustoLogs
| where Timestamp > ago(1h)
| fork
    ( where Level == "Error" | project EventText | limit 100 | as Errors )
    ( project Timestamp, EventText | top 1000 by Timestamp desc | as EventsTexts )
    ( summarize min(Timestamp), max(Timestamp) by ActivityID | as TimeRangePerActivityID )
    
 KustoLogs
| where Timestamp > ago(1h)
| fork
    Errors = ( where Level == "Error" | project EventText | limit 100 )
    EventsTexts = ( project Timestamp, EventText | top 1000 by Timestamp desc )
    TimeRangePerActivityID = ( summarize min(Timestamp), max(Timestamp) by ActivityID )
```