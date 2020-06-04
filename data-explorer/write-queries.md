---
title: Escribir consultas para el Explorador de datos de Azure
description: En estas instrucciones, aprenderá a realizar consultas básicas y avanzadas para el Explorador de datos de Azure.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/07/2019
ms.openlocfilehash: 4490310664a7c27a319ce758a63b1dc90be8bef0
ms.sourcegitcommit: 3cdb365c0174e517fb5cb16784da278ceb801b23
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2020
ms.locfileid: "84214409"
---
# <a name="write-queries-for-azure-data-explorer"></a>Escribir consultas para el Explorador de datos de Azure

En este artículo, aprenderá a usar el lenguaje de consulta en el Explorador de datos de Azure para realizar consultas básicas con los operadores más comunes. También se expondrá a algunas de las características más avanzadas del lenguaje.

## <a name="prerequisites"></a>Requisitos previos

Puede ejecutar las consultas de este artículo de dos maneras:

- En el *clúster de ayuda* del Explorador de datos de Azure que hemos configurado para facilitar el aprendizaje.
    [Inicie sesión en el clúster](https://dataexplorer.azure.com/clusters/help/databases/samples) con una cuenta de correo electrónico organizativa que sea miembro de Azure Active Directory.

- En su propio clúster que incluye los datos de ejemplo de StormEvents. Para más información, consulte [Guía de inicio rápido: Creación de un clúster y de la base de datos de Azure Data Explorer](create-cluster-database-portal.md) e [Ingesta de datos de ejemplo en Azure Data Explorer](ingest-sample-data.md).

    [!INCLUDE [data-explorer-storm-events](includes/data-explorer-storm-events.md)]

## <a name="overview-of-the-query-language"></a>Información general sobre el lenguaje de consulta

Una consulta en Azure Data Explorer es una solicitud de solo lectura para procesar los datos y devolver resultados. La solicitud se indica con un texto sin formato, mediante un modelo de flujo de datos diseñado para simplificar la lectura, creación y automatización de la sintaxis. La consulta usa entidades de esquema cuya organización es una jerarquía similar a SQL: bases de datos, tablas y columnas.

La consulta consta de una secuencia de instrucciones de consulta, delimitada por un punto y coma (`;`), con al menos una instrucción que será una instrucción de expresión tabular, que es una instrucción que genera datos organizados en una malla similar a una tabla de columnas y filas. Las instrucciones de expresión tabular de la consulta generan los resultados de la consulta.

La sintaxis de la instrucción de expresión tabular tiene un flujo de datos tabulares desde un operador de consulta tabular a otro, empezando por el origen de datos (por ejemplo, una tabla en una base de datos o un operador que genera datos) y, después, fluyendo a través de un conjunto de operadores de transformación de datos que están unidos mediante el uso del delimitador de canalización (`|`).

Por ejemplo, la siguiente consulta tiene una sola instrucción, que es una instrucción de expresión tabular. La instrucción se inicia con una referencia a una tabla denominada `StormEvents` (aquí la base de datos que hospeda esta tabla es implícita y forma parte de la información de conexión). Después, los datos (filas) de esa tabla se filtran por el valor de la columna `StartTime` y, luego, según el valor de la columna `State`. Después, la consulta devuelve el recuento de filas que "sobreviven".

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSjPSC1KVQguSSwqCcnMTVWws1VISSxJLQGyNYwMDMx1DQ11DQw1FRLzUpBU2aArMgIpQjGvJFXB1lZByc3HP8jTxVFJQQEkm5xfmlcCAHoR9euCAAAA) **\]**

```Kusto
StormEvents
| where StartTime >= datetime(2007-11-01) and StartTime < datetime(2007-12-01)
| where State == "FLORIDA"  
| count
```

En este caso, el resultado es:

|Count|
|-----|
|   23|
| |

Para más información, consulte la [referencia de lenguaje de consulta](https://aka.ms/kustolangref).

## <a name="most-common-operators"></a>Operadores más comunes

Los operadores tratados en esta sección son los bloques de creación para comprender las consultas en el Explorador de datos de Azure. La mayoría de las consultas que se escriben incluyen algunos de estos operadores.

Para ejecutar consultas en el clúster de ayuda, haga clic en **Haga clic para ejecutar la consulta** encima de cada consulta.

Para ejecutar consultas en su propio clúster:

1. Copie cada consulta en la aplicación de consulta basada en web y, después, seleccione la consulta o coloque el cursor en la consulta.

1. En la parte superior de la aplicación, haga clic en **Ejecutar**.

### <a name="count"></a>count

[**count**](kusto/query/countoperator.md): devuelve el recuento de filas de la tabla.

La siguiente consulta devuelve el recuento de filas de la tabla StormEvents.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVqhRSM4vzSsBALU2eHsTAAAA) **\]**

```Kusto
StormEvents | count
```

### <a name="take"></a>take

[**take**](kusto/query/takeoperator.md): devuelve al número especificado de filas de datos.

La siguiente consulta devuelve cinco filas de la tabla StormEvents. La palabra clave *limit* es un alias para *take.*

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVqhRKEnMTlUwBQDEz2b8FAAAAA%3d%3d) **\]**

```Kusto
StormEvents | take 5
```

> [!TIP]
> No hay ninguna garantía de qué registros se devuelven a menos que se ordenen los datos de origen.

### <a name="project"></a>proyecto

[**project**](kusto/query/projectoperator.md): selecciona un subconjunto de columnas.

La siguiente consulta devuelve un conjunto de columnas específico.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUShJzE5VMAWxCorys1KTSxSCSxKLSkIyc1N1FFzzUiAMoFgJiA%2fSFlJZAGS6JOYmpqcGFOUXpBaVVAKlCjKL81NS%2fRKLihJLMstSAY%2buIINnAAAA) **\]**

```Kusto
StormEvents
| take 5
| project StartTime, EndTime, State, EventType, DamageProperty, EpisodeNarrative
```

### <a name="where"></a>, donde

[**where**](kusto/query/whereoperator.md): Filtra una tabla para el subconjunto de filas que cumplen un predicado.

La siguiente consulta filtra los datos por `EventType` y `State`.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAEWMPQvCMBCGd8H%2fcFuWro4dBOvHkgoJOB%2fm0KjJhetRKfjjNe3g9n49r1OW1I2UdVivPvC%2bkxDM3k%2bFoG3B7F%2fMwQDmAE5Rl%2fCydceTPfjemsopPgk2VRXhB121TkV9TNRAl8MiZrz53zeww4Q3OgsXEp1%2bVYkDB7IoghpH%2bgI9OH8WnwAAAA%3d%3d) **\]**

```Kusto
StormEvents
| where EventType == 'Flood' and State == 'WASHINGTON'
| take 5
| project StartTime, EndTime, State, EventType, DamageProperty, EpisodeNarrative
```

### <a name="sort"></a>sort

[**sort**](kusto/query/sortoperator.md): Ordena las filas de la tabla de entrada en una o más columnas.

La siguiente consulta ordena los datos en orden descendente por `DamageProperty`.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAF2NPQvCMBCGd8H%2fcFuXrI4dBOvHEoUGnM%2fm0KjphctRKfjjNe0guL0fvM%2fbKktsBuo1LxdveN1ICCbvxkRQ11Btn8y%2bAuw9tIo6h%2bd1uz%2fYnTvaquwyi8JlhA1GvNJJOJHoCJ5yV2rFB8GqqCR8p04LSdSFSAaa3s9iopvfu%2fnDfasUMnuyKIIaBvoAtvGMsb4AAAA%3d) **\]**

```Kusto
StormEvents
| where EventType == 'Flood' and State == 'WASHINGTON'
| sort by DamageProperty desc
| take 5
| project StartTime, EndTime, State, EventType, DamageProperty, EpisodeNarrative
```

> [!NOTE]
> Es importante el orden de las operaciones. Intente colocar `take 5` antes de `sort by`. ¿Obtiene resultados diferentes?

### <a name="top"></a>top

[**top**](kusto/query/topoperator.md): Devuelve los primeros *N* registros ordenados por las columnas especificadas.

La consulta siguiente devuelve los mismos resultados que anteriormente con un operador menos.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAF2NOwvCMBSFd8H%2fcLcsWR07CNbHkgoJOMfmohGTG24vlYA%2fXtsOgtt5cL5jhTi1I2YZ1qs3vO7ICLN3tSA0Daj9kygo8DmAFS9LeNna48kcXGfUtBMqsIFrhZ1P%2foZnpoIsFQIO%2fdQXpgf2MgFYXEyooc1hETNU%2f071H%2bRblThQQOOZvcQRP1rSng21AAAA) **\]**

```Kusto
StormEvents
| where EventType == 'Flood' and State == 'WASHINGTON'
| top 5 by DamageProperty desc
| project StartTime, EndTime, State, EventType, DamageProperty, EpisodeNarrative
```

### <a name="extend"></a>extend

[**extend**](kusto/query/extendoperator.md): calcula las columnas derivadas.

La siguiente consulta crea una nueva columna calculando un valor en cada fila.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAF2OvQ7CMAyEdyTewVuWMDJ2QGr5WQJSKzGHxoIiEkeuKVTi4WmooBKbfXeffaUQ%2b6LDIO189oLHBRnhs1d9RMgyUOsbkVNgg4NSrIzicVVud2ZT7Y1KnFCEJZx6yK23ZzwwRWTpwWFbJx%2bfggOf39lKQwEyKIKrGo%2bwSEdZ0pyCkemKtUyi%2fib1j9ZjDz311H9%2fBys2LTk0lhPT4RvwA3pn6AAAAA%3d%3d) **\]**

```Kusto
StormEvents
| where EventType == 'Flood' and State == 'WASHINGTON'
| top 5 by DamageProperty desc
| extend Duration = EndTime - StartTime
| project StartTime, EndTime, Duration, State, EventType, DamageProperty, EpisodeNarrative
```

Las expresiones pueden incluir todos los operadores habituales (+, -, *, /, %) y hay una amplia gama de funciones útiles a las que puede llamar.

### <a name="summarize"></a>summarize

[**summarize**](kusto/query/summarizeoperator.md): incorpora grupos de filas.

La consulta siguiente devuelve el recuento de eventos por `State`.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlVIBYnFJ%2beX5pUo2CqAaQ1NhaRKheCSxJJUAB%2fedDI3AAAA) **\]**

```Kusto
StormEvents
| summarize event_count = count() by State
```

El operador **summarize** agrupa las filas que tienen los mismos valores en el cláusula **by** y, después, usa la función de agregación (como **count**) para combinar cada grupo en una sola fila. Por lo tanto, en este caso, hay una fila para cada estado y una columna para el recuento de filas de ese estado.

Hay una amplia gama de funciones de agregación y puede usar varias en un operador **summarize** para generar varias columnas calculadas. Por ejemplo, podría obtener el recuento de tormentas en cada estado y el número único de tormentas por estado, y luego usar **top** para obtener los estados más afectados por tormentas.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlUIBkk455fmlSjYKiSDaA1NHYWQyoJU%2fzSwXDFQPAUiAdYPktJUSKoE6kwsSQUZVpJfoGAKEYGblZJanAwAgbFb73QAAAA%3d) **\]**

```Kusto
StormEvents
| summarize StormCount = count(), TypeOfStorms = dcount(EventType) by State
| top 5 by StormCount desc
```

El resultado de una operación **summarize** tiene:

- Cada columna con nombre en **by**

- Una columna para cada expresión calculada

- Una fila para cada combinación de valores by

### <a name="render"></a>render

[**render**](kusto/query/renderoperator.md): representa los resultados como una salida gráfica.

La consulta siguiente muestra un gráfico de columnas.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAFWMsQ7CQAxDdyT%2bIWMrdSgbSxmQ2Nj6Aei4Ru0hkqA0VwTi49uUBRZL9rPdmiidJmQbt5sPjJkoaHojoGeXKJmtWbUoK6DUQQNh6osj9onPwUq4vqC1YLjORc2Dpef2OaD%2bPcEBdvu6dvZQuWG077b6LTlV5A4VotwzcRyC2gxU6ktSqQAAAA%3d%3d) **\]**

```Kusto
StormEvents
| summarize event_count=count(), mid = avg(BeginLat) by State
| sort by mid
| where event_count > 1800
| project State, event_count
| render columnchart
```

La consulta siguiente muestra un gráfico de tiempo simple.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlVIBYnFJ%2beX5pXYgkkNTYWkSoWkzDyN4JLEopKQzNxUHQXDFE2QtqLUvJTUIoUSoFhyBlASAAyXWQJWAAAA) **\]**

```Kusto
StormEvents
| summarize event_count=count() by bin(StartTime, 1d)
| render timechart
```

La consulta siguiente cuenta los eventos por el módulo de tiempo de un día, discretizados en horas y muestra un gráfico de tiempo.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEADWNQQqDMBRE90LvMBtBwY0HcNkT2L2k8UuEJh9%2bfqSWHt4k4GZghpk3s7L450FB46P5g75KYYXjJJiwfZilm9WIvnZPaDGuGDC6vnRj8t7I%2fiNQ2S%2bWU9CpatfjfVZKLbLo7WGiLZnkGxJoxlqX%2bRf81ZbyiAAAAA%3d%3d) **\]**

```Kusto
StormEvents
| extend hour = floor(StartTime % 1d , 1h)
| summarize event_count=count() by hour
| sort by hour asc
| render timechart
```

La siguiente consulta compara varias series diarias en un gráfico de tiempo.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEACWPSwvCMBCE74L%2fYSgIFXrpD%2bihaKzxkUBTXyeputKCbSCmvvDHm9TL7gwzsN8qq03DHtTa%2b3DwBb0stRdUujMJrjetTQhlS2OLuiGMEF8QIa7GvvusyJBPLaFuEQbZZjWDnGHN9nwigyhYp1wwt7c8z7jgqZM7riZSKC6cFjIv5pimS1n4SLAdFixX7OCMzFkmRdAfundNU5r6QyAPejzrrrVJP8MxTu8eN%2fqT%2bL5xL5CBdcjnyrH%2fALPTSKnkAAAA) **\]**

```Kusto
StormEvents
| extend hour= floor( StartTime % 1d , 1h)
| where State in ("GULF OF MEXICO","MAINE","VIRGINIA","WISCONSIN","NORTH DAKOTA","NEW JERSEY","OREGON")
| summarize event_count=count() by hour, State
| render timechart
```

> [!NOTE]
> El operador **render** es una característica de lado cliente en lugar de parte del motor. Está integrado en el lenguaje para facilitar su uso. La aplicación web admite las siguientes opciones: barchart, columnchart, piechart, timechart y linechart. 

## <a name="scalar-operators"></a>Operadores escalares

Esta sección trata algunos de los operadores escalares más importantes.

### <a name="bin"></a>bin()

[**bin()** ](kusto/query/binfunction.md): Redondea los valores hacia abajo hasta un entero múltiplo del tamaño de un intervalo determinado.

La siguiente consulta calcula el número con un tamaño de depósito de un día.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSjPSC1KVQguSSwqCcnMTVWwU0hJLEktATI1jAwMzHUNjHQNTTQVEvNSkBTZYCoyMtQEGVdcmpubWJRZlaqQCrIiPjm%2fNK9EwVYBTGtoKiRVKiRl5mnAjdJRMEzRBABIhjnmkwAAAA%3d%3d) **\]**

```Kusto
StormEvents
| where StartTime > datetime(2007-02-14) and StartTime < datetime(2007-02-21)
| summarize event_count = count() by bin(StartTime, 1d)
```

### <a name="case"></a>case()

[**case()** ](kusto/query/casefunction.md): evalúa una lista de predicados y devuelve la primera expresión de resultado cuyo predicado se cumpla o la última expresión **else**. Puede usar este operador para clasificar o agrupar los datos:

La consulta siguiente devuelve una columna `deaths_bucket` nueva y agrupa las muertes por número.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAGWOwQrCQAxE74X%2bQ9hTCwX14FFBaK9e%2bgGS7gZdbFrYZEXFj7dbqgfNbfJmhml1DNzcaFDJsxdIZMbgnwSOUC8Cu%2fQq6lnUPpDVEroHtIpKKUB3pcEt7lMX7ZV0ClkUgiLPYLqlaQ%2fbdQWmx3AmU%2f2gTUJMzkf%2bYwkJY99%2fiDmuDqac545Bv3MAxb4Bic1Oy88AAAA%3d) **\]**

```Kusto
StormEvents
| summarize deaths = sum(DeathsDirect) by State
| extend deaths_bucket = case (
    deaths > 50, "large",
    deaths > 10, "medium",
    deaths > 0, "small",
    "N/A")
| sort by State asc
```

### <a name="extract"></a>extract()

[**extract()** ](kusto/query/extractfunction.md): obtiene una coincidencia para una expresión regular a partir de una cadena determinada.

La siguiente consulta extrae los valores de atributo específicos de un seguimiento.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAE2OwQrCMBBE74X%2bw9BTojHYagSVHJRevXkrHqJdpVBbSVew4McbFYungeXtvKmJsetzxw4WZQh2x5og9t6daIWOfdVcJIpkY1OFrc0U8rt3XLWNTbOZnhultU4UfoD5A4zRmVkovInDOo6%2bojh6gh5MTTmQwR0uQckiGb5FMZ0s9WEsQ3uo%2fixSccT9jdqz8ORqKTECV1cSaSdfq2k6L8oAAAA%3d) **\]**

```Kusto
let MyData = datatable (Trace: string) ["A=1, B=2, Duration=123.45,...", "A=1, B=5, Duration=55.256, ..."];
MyData
| extend Duration = extract("Duration=([0-9.]+)", 1, Trace, typeof(real)) * time(1s)
```

Esta consulta usa una instrucción **let**, la cual enlaza un nombre (en este caso `MyData`) con una expresión. Para el resto del ámbito, en el que aparece la instrucción **let** (ámbito global o en un ámbito del cuerpo de función), se puede usar el nombre para hacer referencia a su valor enlazado.

### <a name="parse_json"></a>parse_json()

[**parse_json()** ](kusto/query/parsejsonfunction.md): interpreta una cadena como un valor JSON y devuelve el valor como dinámico. Es superior al uso de la función **extractjson()** cuando necesita extraer más de un elemento de un objeto JSON compuesto.

La siguiente consulta extrae los elementos JSON de una matriz.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAHWPQQuCQBCF74L%2fYdmLBSJ6EGKjU17r1E0kJh1C2XZlHc0w%2f3ur1s1O896bB%2fONRGKnVwIE7MAKOwhuEtnmYiBHwRoypbpvXSf1Bl60BqjUiot04B3IFrmIol0Q%2bpPLdauIi3iyj9KWojCcNfRWx7NuqEiw48KaMRu9bO86y3HXeTPsCVXBzvg8amlpajANXqtGq4VmO5VqoyvM6dsKfkhpmAUzkf9nM9OtLi3reg79ar788AEVX8GkOAEAAA%3d%3d) **\]**

```Kusto
let MyData = datatable (Trace: string)
['{"duration":[{"value":118.0,"valcount":5.0,"min":100.0,"max":150.0,"stdDev":0.0}]}'];
MyData
| extend NewCol = parse_json(Trace)
| project NewCol.duration[0].value, NewCol.duration[0].valcount, NewCol.duration[0].min, NewCol.duration[0].max, NewCol.duration[0].stdDev
```

La siguiente consulta extrae los elementos JSON.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAE2OwQqCQBCG74LvsOzFBBE9CLHRKa916hYRkw6RbLuyO5pRvXvrGtZpvn9m4P8kEts%2bSiBga1a7QXCWyBZ7AxUKZslc1SVmh%2bjJe5AdcpHnyzRLxlTpThEXxRhvV%2bVOWeYZBseFZ0t1iT0XLryj4yoMprIweDEcCFXNdnjfaOnaWzAWT43VamqPx6fW6AYr%2bn6l3iH5S95hXjiLH8Mw82TxAQvJEB%2fsAAAA) **\]**

```Kusto
let MyData = datatable (Trace: string) ['{"value":118.0,"valcount":5.0,"min":100.0,"max":150.0,"stdDev":0.0}'];
MyData
| extend NewCol = parse_json(Trace)
| project NewCol.value, NewCol.valcount, NewCol.min, NewCol.max, NewCol.stdDev
```

La siguiente consulta extrae los elementos JSON con un tipo de datos dinámico.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAD2NMQvCMBBG90D%2bw5GphVLSoSARt65ubuJwJjdU0lZiWlrU%2f25MotO9x8H7LHk4bh16hAOYcDxeLUFxcqhJgdlGHHpdcnbOWDzFgnYmoZpmV8tK6GkePTmh2q8N%2fRg%2bUkbGNXAb%2beFNR4tQQd7lZc9ZGuXsBXc33Uh7iJN1jFdZcvunIf5HXCvOEqf2BwXmDCnKAAAA) **\]**

```Kusto
let MyData = datatable (Trace: dynamic)
[dynamic({"value":118.0,"counter":5.0,"min":100.0,"max":150.0,"stdDev":0.0})];
MyData
| project Trace.value, Trace.counter, Trace.min, Trace.max, Trace.stdDev
```

### <a name="ago"></a>ago()

[**ago()** ](kusto/query/agofunction.md): Resta un intervalo de tiempo especificado a la hora UTC actual.

La siguiente consulta devuelve los datos de las últimas 12 horas.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA1WOQQ6CQAxF9yTc4S8hQcmQuNSNR4ALTKQyJDAlnSIuPLwzJGrctM3v+7+t684R7qMEhW6MafQUMJAnsUoIdl4mQm/VVrC+h0Z6shFOINZAIc/qOql24KIEL8nIAuWYohC6sfQB9yjtPtPA8SrhmGeLjF7RjTO1Gu+cIdYPVHjeisOpLyukKTbjYml5piuvXknwIU1lGlPm2Qvzg55L+u+b9udIyOZI6LfHZf/YNK58Ay2HrbAEAQAA) **\]**

```Kusto
//The first two lines generate sample data, and the last line uses
//the ago() operator to get records for last 12 hours.
print TimeStamp= range(now(-5d), now(), 1h), SomeCounter = range(1,121)
| mv-expand TimeStamp, SomeCounter
| where TimeStamp > ago(12h)
```

### <a name="startofweek"></a>startofweek()

[**startofweek()** ](kusto/query/startofweekfunction.md): devuelve el inicio de la semana que contiene la fecha, desplazada por un desplazamiento, si se proporciona.

La siguiente consulta devuelve el inicio de la semana con desplazamientos diferentes.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEACtKzEtPVchPSytOLVFIK8rPVdA1VCjJVzBUKC5JLVAw5OWqUSgoys9KTS5RKE9NzQ4uSSwqUbAFygLp%2fDSQkEZefrmGpg7UEE0dCA0AdE3lv1kAAAA%3d) **\]**

```Kusto
range offset from -1 to 1 step 1
| project weekStart = startofweek(now(), offset),offset
```

Esta consulta usa el operador **range**, que genera una tabla de valores de una sola columna. Vea también: [**startofday()** ](kusto/query/startofdayfunction.md), [**startofweek()** ](kusto/query/startofweekfunction.md), [**startofyear()** ](kusto/query/startofyearfunction.md)), [**startofmonth()** ](kusto/query/startofmonthfunction.md), [**endofday()** ](kusto/query/endofdayfunction.md), [**endofweek()** ](kusto/query/endofweekfunction.md), [**endofmonth()** ](kusto/query/endofmonthfunction.md) y [**endofyear()** ](kusto/query/endofyearfunction.md).

### <a name="between"></a>between()

[**between()** ](kusto/query/betweenoperator.md): coincide con la entrada que está dentro del intervalo inclusivo.

La siguiente consulta filtra los datos por un intervalo de fecha determinado.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSjPSC1KVQguSSwqCcnMTVVISi0pT03NU9BISSxJLQGKaBgZGJjrApGRuaaCnp4ChrixgaYmyKTk%2fNK8EgBluyagXgAAAA%3d%3d) **\]**

```Kusto
StormEvents
| where StartTime between (datetime(2007-07-27) .. datetime(2007-07-30))
| count
```

La siguiente consulta filtra los datos según un intervalo de fechas determinado, con una ligera variación de tres días (`3d`) desde la fecha de inicio.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSjPSC1KVQguSSwqCcnMTVVISi0pT03NU9BISSxJLQGKaBgZGJjrApGRuaaCnp6CcYomSF9yfmleCQCGAqjRTAAAAA%3d%3d) **\]**

```Kusto
StormEvents
| where StartTime between (datetime(2007-07-27) .. 3d)
| count
```

## <a name="tabular-operators"></a>Operadores tabulares

Kusto tiene muchos operadores tabulares, algunos de los cuales se tratan en otras secciones de este artículo. Aquí nos centraremos en **parse**. 

### <a name="parse"></a>parse

[**parse**](kusto/query/parseoperator.md): evalúa una expresión de cadena y analiza su valor en una o más columnas calculadas. Hay tres maneras de analizar: simple (el valor predeterminado), regex y relajado.

La siguiente consulta analiza un seguimiento y extrae los valores pertinentes, con un valor predeterminado de un análisis simple. La expresión (denominada StringConstant) es un valor de cadena regular y la coincidencia es estricta: las columnas extendidas deben coincidir con los tipos necesarios.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAN2UTU%2fDMAyG75X6H6xcxlCkpRlsUNQjN6gQ2wnEoevMFsiaKk2HJvHjabqvlI91l11QLrH12vETW5Zo4H411kmKEME0MdWZSISz2yVmpvaHhdEim3V979n3OrU%2fhFgZ8boaSZHiI0pMiipEY6FKnWKcLDB6EDlKkeEoneO0lKgpGGUSWYcUER9SKOw1LhcT1BHvU5AqfR%2bLKpbxXjDscRYMgF2FFyxkwRMFvX7ngCLXuBSqLO5%2bT9S%2ftrJuh54OI7g8iMFaMdhxGOy0GJz9i25w%2fjdG0IoRHNWNNe1ph2pwEKNlqI7HsEPley83vrfZCL73CXmiq%2fr32wA%2bhJnDOZAGEQHXBNIEIq4VSpXNbAIXkbjAO8UOmuz4bWoXlrhWWO0vqyA2%2bAcw2f7B1rORd60calat3jA1TRbq1A6NxsC%2bLdCoCuj3p74AKTs4pmcFAAA%3d) **\]**

```Kusto
let MyTrace = datatable (EventTrace:string)
[
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=23, lockTime=02/17/2016 08:40:01Z, releaseTime=02/17/2016 08:40:01Z, previousLockTime=02/17/2016 08:39:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00Z, releaseTime=02/17/2016 08:40:00Z, previousLockTime=02/17/2016 08:39:00Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01Z, releaseTime=02/17/2016 08:40:01Z, previousLockTime=02/17/2016 08:39:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01Z, releaseTime=02/17/2016 08:41:00Z, previousLockTime=02/17/2016 08:40:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00Z, releaseTime=02/17/2016 08:41:00Z, previousLockTime=02/17/2016 08:40:00Z)'
];
MyTrace
| parse EventTrace with * "resourceName=" resourceName ", totalSlices=" totalSlices:long * "sliceNumber=" sliceNumber:long * "lockTime=" lockTime ", releaseTime=" releaseTime:date "," * "previousLockTime=" previouLockTime:date ")" *  
| project resourceName ,totalSlices , sliceNumber , lockTime , releaseTime , previouLockTime
```

La siguiente consulta analiza un seguimiento y extrae los valores pertinentes mediante `kind = regex`. StringConstant puede ser una expresión regular.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAN2UQU%2fCQBCF7036HyZ7gWKRbVHQmgY9eNPGCCcoh9KOsLK0ZLtFMf54l6LQBgUuXEyTTbP7pt3vvclwlPC47IkgRHAhCqR6Rhyher%2fAWOb7TioFi8eGrg10rZLvO%2bAlkr0su5yF%2bIwcg1SVCEyTTIToBTN0n9gcOYuxG04wyjgKE2QiA56XpK7dNiFdvXrZbITCtZsm8CSc9piqpXbDajdsarWAXjkX1KFW3wSx%2fs8exVzggiVZ%2bvD7h5rXK5lRMU%2bHYV3uxaAHMehxGPS0GDb9F2nY9t8Y1kEM66g01rSnbarWXowDTXU8xqqpdG14o2vfE0HXPmEeCHX%2fKYsjNR8EjvEdtqMB3picAKme1zrGIKh%2f3NX7w5pLoEgLt6SM56c1PzpTq6oqYpIitMOTeAxAlKb6c3Wjs3GBbAzJJUV8UjQjP91BJztuOGryKbHvGwQgxxbJK4ayTFKKBbahQCkA2DX7C29veJJmBQAA) **\]**

```Kusto
let MyTrace = datatable (EventTrace:string)
[
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=23, lockTime=02/17/2016 08:40:01Z, releaseTime=02/17/2016 08:40:01Z, previousLockTime=02/17/2016 08:39:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00Z, releaseTime=02/17/2016 08:40:00Z, previousLockTime=02/17/2016 08:39:00Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01Z, releaseTime=02/17/2016 08:40:01Z, previousLockTime=02/17/2016 08:39:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01Z, releaseTime=02/17/2016 08:41:00Z, previousLockTime=02/17/2016 08:40:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00Z, releaseTime=02/17/2016 08:41:00Z, previousLockTime=02/17/2016 08:40:00Z)'
];
MyTrace
| parse kind = regex EventTrace with "(.*?)[a-zA-Z]*=" resourceName @", totalSlices=\s*\d+\s*.*?sliceNumber=" sliceNumber:long  ".*?(previous)?lockTime=" lockTime ".*?releaseTime=" releaseTime ".*?previousLockTime=" previousLockTime:date "\\)"  
| project resourceName , sliceNumber , lockTime , releaseTime , previousLockTime
```

La siguiente consulta analiza un seguimiento y extrae los valores pertinentes mediante `kind = relaxed`. StringConstant es un valor de cadena regular y la coincidencia es relajada: las columnas extendidas pueden coincidir parcialmente con los tipos necesarios.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAN2US0%2fCQBDH7036HSZ7wZpN2BYFrenRGzZG4KLxUNoRVpYu2W5REj%2b83fKw9QE1kYvppTOZx%2f%2b3MxmBGm5WQxXFCAEkkS6%2bsUA4uV5iqku%2fn2nF04ljWw%2b21Sr9PoRS86fVQPAY71BglBUpCjOZqxjDaI7BLV%2bg4CkO4ikmuUBFQUsdiTIlC7wehcz8hvl8jCrwOhSEjGdDXuQyr%2b322h5zu8Au%2fDPmM%2feeglr32ROxULjkMs%2f63xfqXJowp0WPh%2bGe78VgBzFYMwx2XAyP%2fYtpeN7PGO5BDLfRNNa0x12q7l6MA0vVHMMslW09XtnW5iLY1hssIlXon%2fE0CYom0SsmQP6IMxz1%2b7%2b7AnXQdX6TNXMIvHA9hVMgNYEEqiaQuj5StXwh04kpUNVLqup3ETsCsoMxpavSSdXyi7NrIohJ%2foJDtoRbzybcMeFQjkjJZ4x1nYVWtEPtleHjjaGmCujnVu%2fWU75tHgYAAA%3d%3d) **\]**

```Kusto
let MyTrace = datatable (EventTrace:string)
[
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=23, lockTime=02/17/2016 08:40:01Z, releaseTime=02/17/2016 08:40:01Z, previousLockTime=02/17/2016 08:39:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00Z, releaseTime=02/17/2016 08:40:00Z, previousLockTime=02/17/2016 08:39:00Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01Z, releaseTime=02/17/2016 08:40:01Z, previousLockTime=02/17/2016 08:39:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01Z, releaseTime=02/17/2016 08:41:00Z, previousLockTime=02/17/2016 08:40:01Z)',
'Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00Z, releaseTime=02/17/2016 08:41:00Z, previousLockTime=02/17/2016 08:40:00Z)'
];
MyTrace
| parse kind=relaxed "Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=NULL, sliceNumber=23, lockTime=02/17/2016 08:40:01, releaseTime=NULL, previousLockTime=02/17/2016 08:39:01)" with * "resourceName=" resourceName ", totalSlices=" totalSlices:long * "sliceNumber=" sliceNumber:long * "lockTime=" lockTime ", releaseTime=" releaseTime:date "," * "previousLockTime=" previousLockTime:date ")" *  
| project resourceName ,totalSlices , sliceNumber , lockTime , releaseTime , previousLockTime
```

## <a name="time-series-analysis"></a>Análisis de series temporales

### <a name="make-series"></a>make-series

[**make-series**](kusto/query/make-seriesoperator.md): agrupa los grupos de filas como [summarize](kusto/query/summarizeoperator.md), pero genera un vector de serie (temporal) por cada combinación de valores.

La siguiente consulta devuelve un conjunto de series temporales para el recuento de eventos de tormentas al día. La consulta abarca un período de tres meses para cada estado, y rellena los intervalos que faltan con la constante 0:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUchNzE7VLU4tykwtVsizTc4vzSvR0FRISU1LLM0psTVQyM9TCC5JLCoJycxNVcjMUyhKzEtP1UhJLEktAYpoGBkYmOsaGAKRpo4CmqixrjFI1DBFUyGpEmRKSSoAazsM0n0AAAA%3d) **\]**

```Kusto
StormEvents
| make-series n=count() default=0 on StartTime in range(datetime(2007-01-01), datetime(2007-03-31), 1d) by State
```

Una vez que cree un conjunto de serie (temporal), puede aplicar funciones de la serie para detectar formas anómalas, patrones estacionales y mucho más.

La siguiente consulta extrae los tres estados principales que tenían la mayoría de los eventos en un día específico:

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAF2OsQoCMRBEe8F%2f2DIBAzmvsLrSLzj7EC%2brBs3mSPbkBD%2feLDYibPVmZmdGziUdn0hct5s3JH9HU7FErEDDlBdipSHgxS8PHixkgpF94VNMCJGgeLqiCp6RG1F7aw%2fGdu30Dv5ob3qhXdBwfskXRmnElZECfDtdbbgq0qJwnqEX76%2fmyCW%2ftkV1Ek9pWSwgNdOt7foAJIuybs8AAAA%3d) **\]**

```Kusto
StormEvents
| make-series n=count() default=0 on StartTime in range(datetime(2007-01-01), datetime(2007-03-31), 1d) by State
| extend series_stats(n)
| top 3 by series_stats_n_max desc
| render timechart
```

Para más información, revise la lista completa de [funciones de serie](kusto/query/scalarfunctions.md#series-processing-functions).

## <a name="advanced-aggregations"></a>Agregaciones avanzadas

Se tratan como agregaciones básicas, como **count** y **summarize**, anteriormente en este artículo. Esta sección presenta opciones más avanzadas.

### <a name="top-nested"></a>top-nested

[**top-nested**](kusto/query/topnestedoperator.md): genera los principales resultados jerárquicos, donde cada nivel es un desglose según los valores de niveles anteriores.

Este operador es útil para escenarios de visualización del panel, o cuando es necesario que responda una pregunta similar a la siguiente: "Encontrar los valores principales de N de K1 (mediante la agregación); para cada uno de ellos, buscar cuáles son los valores principales de M de K2 (mediante otra agregación); ..."

La siguiente consulta devuelve una tabla jerárquica con `State` en el nivel superior, seguido por `Sources`.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSjJL9DNSy0uSU1RMFLIT1MILkksSVVIqlQoLs3VcEpNz8zzSSzR1OHlQlJoDFaYX1qUTEilIUila16KT35yYklmfh6GcgDrXwk5jgAAAA%3d%3d) **\]**

```Kusto
StormEvents
| top-nested 2 of State by sum(BeginLat),
top-nested 3 of Source by sum(BeginLat),
top-nested 1 of EndLocation by sum(BeginLat)
```

### <a name="pivot-plugin"></a>complemento pivot()

[**complemento pivot()** ](kusto/query/pivotplugin.md): gira una tabla convirtiendo los valores únicos de una columna de la tabla de entrada en varias columnas de la tabla de salida. El operador realiza agregaciones donde sea necesario en cualquier valor de columna restante en la salida final.

La siguiente consulta aplica un filtro y cambia las filas a columnas.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSgoys9KTS5RCC5JLEnVUQBLhFQWpILkyjNSi1IhMgrFJYlFJcXlmSUZCkqOPkoIabgOhYzEYgWl8My8FLBsalliTilIZ0FmWX6JBtgUTQDlv21NfQAAAA%3d%3d) **\]**

```Kusto
StormEvents
| project State, EventType
| where State startswith "AL"
| where EventType has "Wind"
| evaluate pivot(State)
```

### <a name="dcount"></a>dcount()

[**dcount()** ](kusto/query/dcount-aggfunction.md): devuelve una estimación del número de valores distintos de una expresión en el grupo. Use [**count()** ](kusto/query/countoperator.md) para contar todos los valores.

La consulta siguiente cuenta distintos `Source` por `State`.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlUIzi8tSk4tVrBVSEnOL80r0YAIaCokVSoElySWpAIAFKgSBDoAAAA%3d) **\]**

```Kusto
StormEvents
| summarize Sources = dcount(Source) by State
```

### <a name="dcountif"></a>dcountif()

[**dcountif()** ](kusto/query/dcountif-aggfunction.md): devuelve una estimación del número de valores distintos de la expresión de filas para las que el predicado se evalúa en true.

La siguiente consulta cuenta los valores distintos de `Source` donde `DamageProperty < 5000`.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSspVqhRKEnMTlUwNDDg5apRKC7NzU0syqxKVQjOLy1KTi1WsFVISc4vzSvJTNOACOkouCTmJqanBhTlF6QWlVQq2CiYGhgYaCokVSoElySWpAIAuk%2fTX14AAAA%3d) **\]**

```Kusto
StormEvents 
| take 100
| summarize Sources = dcountif(Source, DamageProperty < 5000) by State
```

### <a name="dcount_hll"></a>dcount_hll()

[**dcount_hll()** ](kusto/query/dcount-hllfunction.md): Calcula el valor de **dcount** de los resultados de HyperLogLog (generados por [**hll**](kusto/query/hll-aggfunction.md) o [**hll_merge**](kusto/query/hll-merge-aggfunction.md).

La consulta siguiente usa el algoritmo HLL para generar el recuento.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlXIyMkJSi1WsAUxNFwScxPTUwOK8gtSi0oqNRWSKhWSMvM0gksSi0pCMnNTdQwNcjUx9PumFqWnpkCMiM8FcTQgpoKVFhTlZ6UmlyikJOeX5pXEg6yB69EEAKm9wyCXAAAA) **\]**

```Kusto
StormEvents
| summarize hllRes = hll(DamageProperty) by bin(StartTime,10m)
| summarize hllMerged = hll_merge(hllRes)
| project dcount_hll(hllMerged)
```

### <a name="arg_max"></a>arg_max()

[**arg_max()** ](kusto/query/arg-max-aggfunction.md): busca una fila en el grupo que maximice una expresión y devuelve el valor de otra expresión (o * para devolver toda la fila).

La consulta siguiente devuelve la hora del último informe de inundación en cada estado.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSjPSC1KVQDzQyoLUhVsbRWU3HLy81OUQLLFpbm5iUWZVakKiUXp8bmJFRrBJYlFJSGZuak6ClqaCkmVCkCBklSQ2oKi%2fKzU5BKIgI4CkkLXvBQoA2YNAHO1S0OFAAAA) **\]**

```Kusto
StormEvents
| where EventType == "Flood"
| summarize arg_max(StartTime, *) by State
| project State, StartTime, EndTime, EventType
```

### <a name="makeset"></a>makeset()

[**makeset()** ](kusto/query/makeset-aggfunction.md): devuelve una matriz dinámica (JSON) del conjunto de valores distintos que una expresión toma en el grupo.

La consulta siguiente devuelve todas las veces que se informó de una inundación en cada estado y crea una matriz del conjunto de valores distintos.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAFWLQQ6CQBAE7yb8ocNJE76wR3mA8IEFOxF1mM3siIHweAVPHqsq1bianCeOnovDiveNRuzczokIAWX9VL2WW80vkWjDQuzuwqTmGQESH8z0Y%2bPRvB2EJ3QzvuTcvmR6Z%2b8%2fUf3NH6ZkMFeAAAAA) **\]**

```Kusto
StormEvents
| where EventType == "Flood"
| summarize FloodReports = makeset(StartTime) by State
| project State, FloodReports
```

### <a name="mv-expand"></a>mv-expand

[**mv-expand**](kusto/query/mvexpandoperator.md): expande las colecciones de varios valores de una columna de tipo dinámico para que cada valor de la colección obtenga una fila independiente. Todas las demás columnas de una fila expandida se duplican. Es lo contrario de makelist.

La siguiente consulta genera datos de ejemplo creando un conjunto y, después, usándolo para demostrar las capacidades de **mv-expand**.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAFWOQQ6CQAxF9yTcoWGliTcws1MPIFygyk9EKTPpVBTj4Z2BjSz%2f738v7WF06r1vD2xcp%2bCoNq9yHDFYLIsvvW5Q0JybKYCco2omqnyNTxHW7oPFckbwajFZhB%2bIsE1trNZ0gi1dpuRmQ%2baC%2bjuuthS7Fbwvi%2f%2bP8lpGvAMP7Wr3A6BceSu7AAAA) **\]**

```Kusto
let FloodDataSet = StormEvents
| where EventType == "Flood"
| summarize FloodReports = makeset(StartTime) by State
| project State, FloodReports;
FloodDataSet
| mv-expand FloodReports
```

### <a name="percentiles"></a>percentiles()

[**percentiles()** ](kusto/query/percentiles-aggfunction.md): devuelve una estimación para el [**percentil del intervalo más cercano**](kusto/query/percentiles-aggfunction.md) especificado de la población definida por una expresión. La precisión depende de la densidad de población en la región del percentil. Se puede usar únicamente en el contexto de agregación dentro de [**summarize**](kusto/query/summarizeoperator.md).

La siguiente consulta calcula los percentiles de duración de tormentas.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUUitKEnNS1FIKS1KLMnMz1OwVXDNSwnJzE1V0FUILkksKgGxQQrLM1KLUhHq7BQMirEI2ygYZ4CEi0tzcxOLMqtSFQpSi5KBlmXmpBZrwJTpKJjqKBgZACkgtgBiS1NNAEC7XiaYAAAA) **\]**

```Kusto
StormEvents
| extend duration = EndTime - StartTime
| where duration > 0s
| where duration < 3h
| summarize percentiles(duration, 5, 20, 50, 80, 95)
```

La siguiente consulta calcula los percentiles de duración de tormentas por estado y normaliza los datos en intervalos de cinco minutos (`5m`).

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAG1NSwrCMBTcC95hli1EKEpBQd31BHUvafOgAZNI8uIPD28SEBVcDDMM8%2bnZedNdyHKYz56gG5NVUNFL1s5ih86qgzaEBXqWnrPOwetEnj65PZrwx95iNWU7RGOk1w8C5avj6KLlNF64qjHcMWhbvXsCralFPmT6rZ%2fJj2lAnyh8pwWWTaKEdcKmLYul%2fgLODFs%2b4AAAAA%3d%3d) **\]**

```Kusto
StormEvents
| extend duration = EndTime - StartTime
| where duration > 0s
| where duration < 3h
| summarize event_count = count() by bin(duration, 5m), State
| summarize percentiles(duration, 5, 20, 50, 80, 95) by State
```

### <a name="cross-dataset"></a>Conjunto de datos cruzado

Esta sección trata los elementos que le permiten crear consultas más complejas, combinar los datos entre tablas y consultas entre bases de datos y clústeres.

### <a name="let"></a>let

[**let**](kusto/query/letstatement.md): mejora la modularidad y la reutilización. La instrucción **let** le permite dividir una expresión potencialmente compleja en varias partes, cada una enlazada a un nombre y recomponer dichas partes. Una instrucción **let** también puede usarse para crear funciones y vistas definidas por el usuario (expresiones a través de las tablas cuyos resultados tienen el aspecto de una tabla nueva). Las expresiones enlazadas por una instrucción **let** puede ser de tipo escalar, de tipo tabular o una función definida por el usuario (expresiones lambda).

El ejemplo siguiente crea una variable de tipo tabular y la usa en una expresión posterior.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAMtJLVHwyUzPKMnLzEsPLskvyi1WsOXlArNcy1LzSop5uWoUyjNSi1IVwPyQyoJUBVtbBSW4LiVrXq4coDGOZYk5iXnJGakkGQPXBTIGzSUgPVn5mXkKGmhmayrk5ykElySWpIKUpGQWl2TmJZdARACul3kY0gAAAA%3d%3d) **\]**

```Kusto
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

### <a name="join"></a>join

[**join**](kusto/query/joinoperator.md): combina las filas de dos tablas para formar una nueva tabla haciendo coincidir los valores de las columnas especificadas de cada tabla. Kusto admite una amplia gama de tipos de combinaciones: **fullouter**, **inner**, **innerunique**, **leftanti**, **leftantisemi**, **leftouter**, **leftsemi**, **rightanti**, **rightantisemi**, **rightouter** y **rightsemi**.

El ejemplo siguiente combina dos tablas con una combinación interna.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA8tJLVGIULBVSEksAcKknFQN79RKq+KSosy8dB2FsMSc0lRDq5z8vHRNXq5oXi4FIFBPVNcx1IGyk9R1jJDYxjB2srqOCS9XrDUvVw7Qhkj8Nhih2wA0ydAAySgjZI4xnJMCtMQAYkuEQo1CVn5mnkJ2Zl6KbWZeXmoR0Nb8PAWgZQAFPLdO5AAAAA==) **\]**

```Kusto
let X = datatable(Key:string, Value1:long)
[
    'a',1,
    'b',2,
    'b',3,
    'c',4
];
let Y = datatable(Key:string, Value2:long)
[
    'b',10,
    'c',20,
    'c',30,
    'd',40
];
X 
| join kind=inner Y on Key
```

> [!TIP]
> Use los operadores **where** y **project** para reducir el número de filas y columnas en las tablas de entrada antes de la combinación. Si una tabla siempre es menor que la otro, úsela como lado izquierdo (canalizado) de la combinación. Las columnas para la coincidencia de la combinación tienen que tener el mismo nombre. Use el operador **project** si es necesario cambiar el nombre de una columna en una de las tablas.

### <a name="serialize"></a>serialize

[**serialize**](kusto/query/serializeoperator.md): serializa el conjunto de filas para que pueda usar las funciones que requieren datos serializados, como **row_number()** .

La siguiente consulta se realiza correctamente porque los datos están serializados.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlVIzi%2fNK9HQVEiqVAguSSxJBcumFmUm5gBlQZzUipLUvBSFovzy%2bLzS3KTUIgVbJI6GJgB4pV4NWgAAAA%3d%3d) **\]**

```Kusto
StormEvents
| summarize count() by State
| serialize
| extend row_number = row_number()
```

El conjunto de filas también se considera como serializado si es el resultado de: operadores **sort**, **top** o **range**, seguidos opcionalmente por operadores **project**, **project-away**, **extend**, **where**, **parse**, **mv-expand** o **take**.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAAsuyS%2fKdS1LzSsp5uWqUSguzc1NLMqsSlVIzi%2fNK9HQVEiqVAguSSxJBcvmF5XABRQSi5NBgqkVJal5KQpF%2beXxeaW5SalFCrZIHA1NAEGimf5iAAAA) **\]**

```Kusto
StormEvents
| summarize count() by State
| sort by State asc
| extend row_number = row_number()
```

### <a name="cross-database-and-cross-cluster-queries"></a>Consultas entre bases de datos y entre clústeres

[Consultas entre bases de datos y entre clústeres](kusto/query/cross-cluster-or-database-queries.md): Puede consultar una base de datos en el mismo clúster haciendo referencia a él como `database("MyDatabase").MyTable`. Puede consultar una base de datos en un clúster remoto haciendo referencia a él como `cluster("MyCluster").database("MyDatabase").MyTable`.

Se llama a la consulta siguiente desde un clúster y consulta los datos del clúster `MyCluster`. Para ejecutar esta consulta, use su propio nombre de clúster y nombre de base de datos.

```Kusto
cluster("MyCluster").database("Wiki").PageViews
| where Views < 2000
| take 1000;
```

### <a name="user-analytics"></a>Análisis de usuario

Esta sección incluye elementos y consultas que muestran lo fácil que es realizar un análisis de comportamientos de usuarios en Kusto.

### <a name="activity_counts_metrics-plugin"></a>complemento activity_counts_metrics

[**complemento activity_counts_metrics**](kusto/query/activity-counts-metrics-plugin.md): calcula las métricas de actividad útiles (valores de recuento total, valores de recuento distintos, recuento distinto de valores nuevos y recuento distinto agregado). Las métricas se calculan para cada ventana temporal y luego se comparan y se agregan y con todas las ventanas temporales.

La siguiente consulta analiza la adopción por parte del usuario calculando los recuentos de actividad diaria.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAJXSPQvCMBAG4L3Q%2f5CtFlLoFyiVDn4M6mqdREpsggTaKs1VEfzxXm0LDiEimcJz3CW8VwogClgDKWcgQFZiEvrB1PNnnh%2b4c9sqsUDUXMPxyA9Z8%2bsjDfhwz0hKsBzPuRSTgxLNlicKGllfKMmwBw6sbsnY0bWto205C4cS3Rso2tpgO4MtDbbSWvixzGD6eb1ttBYZev42%2fbzI8L%2fe9n9b3NkJQ8xs60XEnZUt1hBWgLxLeObFta1B5ZXAKAs1BPuVKO03iXb7gp36tXDfExVB%2f2ICAAA%3d) **\]**

```Kusto
let start=datetime(2017-08-01);
let end=datetime(2017-08-04);
let window=1d;
let T = datatable(UserId:string, Timestamp:datetime)
[
'A', datetime(2017-08-01),
'D', datetime(2017-08-01),
'J', datetime(2017-08-01),
'B', datetime(2017-08-01),
'C', datetime(2017-08-02),
'T', datetime(2017-08-02),
'J', datetime(2017-08-02),
'H', datetime(2017-08-03),
'T', datetime(2017-08-03),
'T', datetime(2017-08-03),
'J', datetime(2017-08-03),
'B', datetime(2017-08-03),
'S', datetime(2017-08-03),
'S', datetime(2017-08-04),
];
T
| evaluate activity_counts_metrics(UserId, Timestamp, start, end,
window)
```

### <a name="activity_engagement-plugin"></a>complemento activity_engagement

[**complemento activity_engagement**](kusto/query/activity-engagement-plugin.md): calcula la proporción de interacción de actividad según la columna de identificador a través de una ventana deslizante de escala de tiempo. El **complemento activity_engagement** puede usarse para calcular los DAU, WAU y MAU (usuarios activos diariamente, semanalmente y mensualmente).

La consulta siguiente devuelve la proporción de usuarios distintos totales que usan una aplicación en comparación con los usuarios distintos totales que usan la aplicación cada semana, en una ventana móvil de siete días.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAG1RQWrDMBC8G%2fyHvUVOHGy1lByKD6GBviDkUIoR1tpVsS0jr0MCeXxXiigpVAiBVjOzM6uigHcc0SlCcGrUdgCtSIFtYZnRgWrInA0ZnNOkR4J6JuUIKo9CMgOKp1LutqXknb1GDI76P8RzQHCXDqHW6gqt43ZRkeydNxNOIHWa3AAv5Ctei2xvx06IQNtGTlZInT0AHQN9BpFt5EO59kHmKvQVUUivX8q1y3L4c9%2fIks%2bt5LoMwsMZLxMrgtHVXcb7pOuEthWemEFvBkPARL%2fSpCjgTfXN0vuBHvbH4rQ%2fsikyNjg6q37xL3GsV47cqQ4HHEl8rIxefeZhNHmMmIehsB2dp8nunnZy9hsbiriDWuqTWqpfxdBsLb2ZGzhm8y%2f6b2i%2bWO8HLFcMGe8BAAA%3d) **\]**

```Kusto
// Generate random data of user activities
let _start = datetime(2017-01-01);
let _end = datetime(2017-01-31);
range _day from _start to _end step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+100*r-1), 1)
| mv-expand id=_users to typeof(long) limit 1000000
// Calculate DAU/WAU ratio
| evaluate activity_engagement(['id'], _day, _start, _end, 1d, 7d)
| project _day, Dau_Wau=activity_ratio*100
| render timechart
```

> [!TIP]
> Al calcular los DAU o MAU, cambian los datos finales y el período de ventana móvil (OuterActivityWindow).

### <a name="activity_metrics-plugin"></a>complemento activity_metrics

[**complemento activity_metrics**](kusto/query/activity-metrics-plugin.md): calcula las métricas de actividad útiles (valores de recuento distintos, recuento distinto de valores nuevos, tasa de retención y tasa de renovación) según la ventana del período actual frente a la ventana del período anterior.

La siguiente consulta calcula la tasa de renovación y retención para un determinado conjunto de datos.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAG2SwW7CMAyG70i8g2%2bk0KoNE%2bIwscsOe4hpqqLGQFjaVKkLVNrDLw7RxjRyqBTr%2fz%2f3t1OW8IYdekUIXnXataAVKXB7GAf0oBoyZ0MGh%2fnMIkE9kPIEO1YhmRbFupLbopJFtc6ekwY7%2fV%2bxKZ4kK0KXA0Kt1QR7H9olIrmbbyDsQer57AvwSlxhFjnruoMQ0VYkT1ZKnd0JfRByBpGt5F255iDDLvYVCaSXm2rpsxz%2b3FfrKnwLGeoygtszXvtABKN3Nwz%2fJ009ur1gYwbWtIZAVvGw53JEn%2fK9PJwSi3rvTthQlOWBPp%2bVJbwq24yWN3FB%2fLQTeAwByLgOeD8x0lnZkRVpL1PdInnTDOJ9YfTiI0%2fE24DyONIctvpB0x94zfBlSJBDcxz97509PgDCM%2bAMzTEgvwEO44wSMAIAAA%3d%3d) **\]**

```Kusto
// Generate random data of user activities
let _start = datetime(2017-01-02);
let _end = datetime(2017-05-31);
range _day from _start to _end step 1d
| extend d = tolong((_day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1)
| mv-expand id=_users to typeof(long) limit 1000000
| where _day > datetime(2017-01-02)
| project _day, id
// Calculate weekly retention rate
| evaluate activity_metrics(['id'], _day, _start, _end, 7d)
| project _day, retention_rate*100, churn_rate*100
| render timechart
```

### <a name="new_activity_metrics-plugin"></a>complemento new_activity_metrics

[**complemento new_activity_metrics**](kusto/query/new-activity-metrics-plugin.md): calcula las métricas de actividad útiles (valores de recuento distintos, recuento distinto de valores nuevos, tasa de retención y tasa de renovación) para la cohorte de usuarios nuevos. El concepto de este complemento es similar a [**complemento activity_metrics**](kusto/query/activity-metrics-plugin.md), pero se centra en los nuevos usuarios.

La siguiente consulta calcula una tasa de retención y renovación con una ventana de semana tras semana para la cohorte de nuevos usuarios (usuarios que llegaron la primera semana).

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAG1Ry27DIBC8W%2fI%2f7C04wbJJFeVQ5VapP9BbVVnIrGMaGyy8eVjqxxcwh1QqBx7LzCwzVBW8o0EnCcFJo%2bwISpIE28F1RgeyJX3TpHHOswEJmpmkIzgFFJIeke1rcSzrQ1mL4jVh0Kj%2fEC8R4bucEd7kAp3z3ZIg2ZU2E04gVJ79AD4oVIIU2cGaM2OBVSZKUQlVPOGcxwUHrNiJp3ITbMyn2JUlHbU91FtXcPhz3u1rP5fC10UUHm%2f4mLwiaHVaZcIzaZnQdiwQCxj0qAlEHUeeVRV8yAuCNcMC1CN02s0Ed8QLtLa33igbpK9M0skRCd3q4CaHa%2fgBg%2fcmJb40%2ft7pdmafG602XzxExpN3HsPicFQ8z1IcQWhy9htbisk2EU92XZ1vZkhb04Sv5tD2V7fufwFYtolnAgIAAA%3d%3d) **\]**

```Kusto
// Generate random data of user activities
let _start = datetime(2017-05-01);
let _end = datetime(2017-05-31);
range Day from _start to _end step 1d
| extend d = tolong((Day - _start)/1d)
| extend r = rand()+1
| extend _users=range(tolong(d*50*r), tolong(d*50*r+200*r-1), 1)
| mv-expand id=_users to typeof(long) limit 1000000
// Take only the first week cohort (last parameter)
| evaluate new_activity_metrics(['id'], Day, _start, _end, 7d, _start)
| project from_Day, to_Day, retention_rate, churn_rate
```

### <a name="session_count-plugin"></a>complemento session_count

[**complemento session_count**](kusto/query/session-count-plugin.md): calcula el recuento de sesiones según la columna de identificador en una escala de tiempo.

La consulta siguiente devuelve el recuento de sesiones. Una sesión se considera activa si un identificador de usuario aparece al menos una vez en un período de tiempo 100 espacios de tiempo, mientras que la ventana de retroceso de sesión es 41 espacios de tiempo.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAFWPQYvCQAyF74X%2bh3dZUCjYgfUgMkcP3r2XoZPqaM3INK4u7I%2ffzOwiNQRC8pKPl5EEnXfiYJEcHwmHcKUxMGFI8QoDidhoYBK6wdTVD%2bgpxB5dd6FvPSuzcwyMS2BvAzMlLP5gez%2fDrNt%2fCN4Z1iwRua5Kk2GPE6WZkY%2bMsRZt1m4pnqmXl9qouK2r1Qo75cUB5RlPQ%2bAgoWDzpPj%2bcuPdCWGiaVKp6%2bOdZbH3zYxmNFuNUhp8mmU%2bTWpWv8or%2fckl%2bQXutT48NwEAAA%3d%3d) **\]**

```Kusto
let _data = range Timeline from 1 to 9999 step 1
| extend __key = 1
| join kind=inner (range Id from 1 to 50 step 1 | extend __key=1) on __key
| where Timeline % Id == 0
| project Timeline, Id;
// End of data definition
_data
| evaluate session_count(Id, Timeline, 1, 10000, 100, 41)
| render linechart
```

### <a name="funnel_sequence-plugin"></a>complemento funnel_sequence

[**complemento funnel_sequence**](kusto/query/funnel-sequence-plugin.md): calcula el recuento distinto de usuarios que han adoptado una secuencia de estados; muestra la distribución de los estados anteriores y siguientes a los que ha llevado la secuencia o que esta ha seguido.

La consulta siguiente muestra qué evento se produce antes y después de todos los eventos de tornados de 2007.

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAEAGWOPYvCQBCG%2b0D%2bw3QmEEmieIqNVQrBRgxYiMhcdqKLyWzcnQiCP95V70DuYIrh5Xk%2f0hRWxpw1H8EwbMTYtrgSiwMnKNqJrtw8DNIU1vkcticUOGHXETv4ptpYgtJYRmWAnrbFGx39QbEWsv%2fIj7YwuHsZmx6FoO6ZqTk4uvTEFUVFp51RtFSJH4hWSt1SAsqj4r9olGXTYZb7i5Mw%2bJRnvzLkKhl%2fTXzAq668dc%2bAG2Orq2g3%2bBk22MfxA23MLGQQAQAA) **\]**

```Kusto
// Looking on StormEvents statistics:
// Q1: What happens before Tornado event?
// Q2: What happens after Tornado event?
StormEvents
| evaluate funnel_sequence(EpisodeId, StartTime, datetime(2007-01-01), datetime(2008-01-01), 1d,365d, EventType, dynamic(['Tornado']))
```

### <a name="funnel_sequence_completion-plugin"></a>complemento funnel_sequence_completion

[**complemento funnel_sequence_completion**](kusto/query/funnel-sequence-completion-plugin.md): calcula el embudo de los pasos de la secuencia completados dentro de diferentes períodos de tiempo.

La consulta siguiente comprueba el embudo de finalización de la secuencia: `Hail -> Tornado -> Thunderstorm -> Wind` en momentos "generales" de una hora, cuatro horas y un día (`[1h, 4h, 1d]`).

**\[** [**Haga clic para ejecutar la consulta**](https://dataexplorer.azure.com/clusters/help/databases/Samples?query=H4sIAAAAAAAAA12QTYvCMBCG74L/YW6tkIV2XT9g8SjsnlvwICKhM9JAOqlJqrj4402CW0RIIB/PPLwzmjwcnZfWwwZQevKqo/yzKFYfRRnW7Hs60ZEhxjdi/UZcFaO5VuqPAjhfLvD/w9F5IG7iM95YdqrJ99mPVDoTkNXGskSTju3ASNZ5Y7t43wVhdhj9PVll0L1aylbAV9glJqyKldsLsXfTyR3oIvUQAsNpYCY95jg2puuDUhnOt71yBukXBVRxCnVoTjwnIlLX4rUzAUlf3/pEPYViDDd7AOyqowFQAQAA) **\]**

```Kusto
let _start = datetime(2007-01-01);
let _end = datetime(2008-01-01);
let _windowSize = 365d;
let _sequence = dynamic(['Hail', 'Tornado', 'Thunderstorm', 'Wind']);
let _periods = dynamic([1h, 4h, 1d]);
StormEvents
| evaluate funnel_sequence_completion(EpisodeId, StartTime, _start, _end, _windowSize, EventType, _sequence, _periods)
```

## <a name="functions"></a>Functions

Esta sección trata las [**funciones**](kusto/query/functions/index.md): consultas reutilizables que se almacenan en el servidor. Las consultas y otras funciones pueden invocar funciones (no se admiten las funciones recursivas).

> [!NOTE]
> No puede crear funciones en el clúster de ayuda, que es de solo lectura. Use su propio clúster de prueba para esta parte.

En el ejemplo siguiente se crea una función que toma un nombre de estado (`MyState`) como argumento.

```Kusto
.create function with (folder="Demo")
MyFunction (MyState: string)
{
StormEvents
| where State =~ MyState
}
```

El ejemplo siguiente llama a una función que obtiene los datos del estado de Texas.

```Kusto
MyFunction ("Texas")
| summarize count()
```

El ejemplo siguiente elimina la función que se creó en el primer paso.

```Kusto
.drop function MyFunction
```

## <a name="next-steps"></a>Pasos siguientes

[Referencia de idioma de consulta de Kusto](https://aka.ms/kustolangref)
