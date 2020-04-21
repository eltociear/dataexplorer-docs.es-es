---
title: 'Límites de consultas: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen los límites de consulta en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/12/2020
ms.openlocfilehash: 437e9781b9e29db7496292ba6a416f6e0c769e8b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523076"
---
# <a name="query-limits"></a>Límites de la consulta

Dado que Kusto es un motor de consultas ad hoc que hospeda grandes conjuntos de datos e intenta satisfacer las consultas manteniendo todos los datos relevantes en memoria, existe un riesgo inherente de que las consultas monopolicen los recursos de servicio sin límites. Kusto proporciona una serie de protecciones integradas en forma de límites de consulta predeterminados.

## <a name="limit-on-query-concurrency"></a>Límite de simultaneidad de consultas

**La simultaneidad** de consultas es un límite que un clúster impone a varias consultas que se ejecutan al mismo tiempo.
El valor predeterminado del límite de simultaneidad de la consulta depende del clúster `Cores-Per-Node x 10`de SKU en el que se ejecuta y se calcula como: . Por ejemplo, para un clúster que está configurado en la SKU de D14v2, donde cada equipo `16 cores x10 = 160`tiene 16 núcleos virtuales, el límite de simultaneidad de consulta predeterminado será .
El valor predeterminado se puede cambiar creando un vale de soporte técnico. En el futuro, este control también se expondrá a través de un comando de control.

## <a name="limit-on-result-set-size-result-truncation"></a>Límite en el tamaño del conjunto de resultados (truncamiento del resultado)

**El truncamiento** de resultados es un límite establecido de forma predeterminada en el conjunto de resultados devuelto por la consulta. Kusto limita el número de registros devueltos al cliente a **500.000**, y la memoria general de esos registros a **64 MB.** Cuando se supera cualquiera de estos límites, se produce un error en la consulta con un "error de consulta parcial". Si se supera la memoria general, se generará una excepción con el mensaje:

```
The Kusto DataEngine has failed to execute a query: 'Query result set has exceeded the internal data size limit 67108864 (E_QUERY_RESULT_SET_TOO_LARGE).'
```

Si se supera el número de registros, se producirá un error con una excepción que dice:

```
The Kusto DataEngine has failed to execute a query: 'Query result set has exceeded the internal record count limit 500000 (E_QUERY_RESULT_SET_TOO_LARGE).'
```

Hay una serie de estrategias para lidiar con este error:

* Reducir el tamaño del conjunto de resultados modificando la consulta para que no devuelva datos poco interesantes. Esto suele ser útil cuando la consulta inicial (que falla) es demasiado "amplia" (por ejemplo, no proyecta columnas de datos que no son necesarias.)
* Reducir el tamaño del conjunto de resultados cambiando el procesamiento posterior a la consulta (como agregaciones) en la propia consulta. Esto es útil en escenarios donde la salida de la consulta se alimenta a otro sistema de procesamiento que, a continuación, realiza agregaciones adicionales. 
* Cambiar de consultas a utilizar la exportación de [datos.](../management/data-export/index.md)
   Esto es adecuado cuando desea exportar grandes conjuntos de datos desde el servicio.
* Indique al servicio que suprima este límite de consultas.

Los métodos comunes para reducir el tamaño del conjunto de resultados generado por la consulta son:

* Mediante el grupo de [operadores](../query/summarizeoperator.md) de resumen y el agregado sobre registros similares en la salida de la consulta, potencialmente muestrear algunas columnas mediante la [función de agregación.](../query/any-aggfunction.md)
* Uso de un [operador take](../query/takeoperator.md) para muestrear la salida de la consulta.
* Uso de la [función de subcadena](../query/substringfunction.md) para recortar columnas de texto libre anchas.
* Usar el operador del [proyecto](../query/projectoperator.md) para quitar cualquier columna poco interesante del conjunto de resultados.

Se puede deshabilitar el truncamiento de resultados mediante la `notruncation` opción de solicitud. Se recomienda encarecidamente que, en este caso, todavía se haya puesto en marcha algún tipo de limitación. Por ejemplo:

```kusto
set notruncation;
MyTable | take 1000000
```

También es posible tener un control más refinado sobre `truncationmaxsize` el truncamiento de resultados estableciendo el valor `truncationmaxrecords` de (tamaño máximo de datos en bytes, valores predeterminados en 64 MB) y (número máximo de registros, valor predeterminado a 500.000). Por ejemplo, la siguiente consulta establece que el truncamiento de resultados se produzca en 1.105 registros o 1 MB, lo que se supere:

```kusto
set truncationmaxsize=1048576;
set truncationmaxrecords=1105;
MyTable | where User=="Ploni"
```

Las bibliotecas de cliente de Kusto asumen actualmente la existencia de este límite. Aunque puede aumentar el límite sin límites, finalmente alcanzará los límites de cliente que actualmente no son configurables. Una posible solución alternativa es programar directamente en el contrato de la API de REST e implementar un analizador de streaming para los resultados de la consulta Kusto. Informe al equipo de Kusto si se encuentra con este problema para que podamos priorizar un cliente de streaming de forma adecuada.

El truncamiento de resultados se aplica de forma predeterminada, no solo a la secuencia de resultados devuelta al cliente; también se aplica de forma predeterminada a cualquier subconsulta que un clúster de Kusto emite a otro clúster de Kusto en una consulta entre clústeres, con efectos similares.

## <a name="limit-on-memory-per-iterator"></a>Límite de memoria por iterador

**La memoria máxima por iterador de conjunto** de resultados es otro límite utilizado por Kusto para proteger contra consultas "descapadas". Este límite (representado por `maxmemoryconsumptionperiterator`la opción de solicitud) establece un límite superior en la cantidad de memoria que puede contener un iterador de conjunto de resultados de plan de consulta único. (Este límite se aplica a los iteradores específicos que no se transmiten por naturaleza, como `join`.) Aquí están algunos mensajes de error que serán devueltos cuando esto suceda:

```
The ClusterBy operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete E_RUNAWAY_QUERY.

The DemultiplexedResultSetCache operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).

The ExecuteAndCache operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).

The HashJoin operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).

The Sort operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).

The Summarize operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).

The TopNestedAggregator operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).

The TopNested operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete (E_RUNAWAY_QUERY).
```

De forma predeterminada, este valor se establece en 5 GB. Uno puede aumentar este valor hasta la mitad de la memoria física de la máquina:

```kusto
set maxmemoryconsumptionperiterator=68719476736;
MyTable | ...
```

Al considerar la eliminación de estos límites, primero determine si realmente obtiene algún valor al hacerlo. En particular, quitar el límite de truncamiento de resultados significa que tiene la intención de mover datos masivos fuera de Kusto, ya sea para fines de exportación (en cuyo caso se le insta a usar el `.export` comando en su lugar) o para realizar la agregación posterior (en cuyo caso, considere la posibilidad de agregar mediante Kusto).
Informe al equipo de Kusto si tiene un escenario empresarial que no puede cumplir con ninguna de estas soluciones sugeridas.  

En muchos casos, se puede evitar exceder este límite mediante el muestreo del conjunto de datos en 10%. Las dos consultas siguientes muestran cómo realizar este muestreo. El primero es un muestreo estadístico (usando un generador de números aleatorios). El segundo es el muestreo determinista (mediante el hash de alguna columna del conjunto de datos, por lo general algún IDENTIFICADOR):

```kusto
T | where rand() < 0.1 | ...

T | where hash(UserId, 10) == 1 | ...
```

## <a name="limit-on-memory-per-node"></a>Límite de memoria por nodo

**La memoria máxima por consulta por nodo** es otro límite utilizado por Kusto para proteger contra consultas "descubiertas". Este límite (representado por `max_memory_consumption_per_query_per_node`la opción de solicitud) establece un límite superior en la cantidad de memoria que se puede asignar en un único nodo para una consulta específica. 


```kusto
set max_memory_consumption_per_query_per_node=68719476736;
MyTable | ...
```

## <a name="limit-on-accumulated-string-sets"></a>Límite en conjuntos de cuerdas acumuladas

En varias operaciones de consulta, Kusto necesita "recopilar" valores de cadena y almacenarlos en búfer internamente antes de que pueda empezar a producir resultados. Estos conjuntos de cadenas acumulados tienen un tamaño limitado y en cuántos elementos pueden contener. Además, cada cadena individual no puede superar un determinado límite.
Si se supera uno de estos límites, se producirá uno de los siguientes errores:

```
Runaway query (E_RUNAWAY_QUERY). (message: 'Accumulated string array getting too large and exceeds the limit of ...GB (see https://aka.ms/kustoquerylimits)')

Runaway query (E_RUNAWAY_QUERY). (message: 'Accumulated string array getting too large and exceeds the maximum count of 2G items (see http://aka.ms/kustoquerylimits)')

Runaway query (E_RUNAWAY_QUERY). (message: 'Single string size shouldn't exceed the limit of 2GB (see http://aka.ms/kustoquerylimits)')
```

Actualmente no hay ningún modificador para aumentar el tamaño máximo del conjunto de cadenas.
Como solución alternativa, reformule la consulta para reducir la cantidad de datos que se deben almacenar en búfer. Por ejemplo, proyectando columnas innecesarias antes de que "introduzcan" operadores como join y summarize. O, por ejemplo, mediante la estrategia de [consulta aleatoria.](../query/shufflequery.md)

## <a name="limit-on-request-execution-time-timeout"></a>Límite en el tiempo de ejecución de la solicitud (tiempo de espera)

**El tiempo** de espera del servidor es un tiempo de espera del lado del servicio que se aplica a todas las solicitudes.
El tiempo de espera en las solicitudes de ejecución (consultas y comandos de control) se aplica en varios puntos:

* En la biblioteca de cliente de Kusto (si se utiliza)
* En el punto de conexión de servicio de Kusto que acepta la solicitud
* En el motor de servicio Kusto que procesa la solicitud

De forma predeterminada, el tiempo de espera se establece en cuatro minutos para las consultas y diez minutos para los comandos de control. Este valor se puede aumentar si es necesario (con un límite de una hora):

* Si va a realizar consultas con Kusto.Explorer, utilice Tiempo de espera del servidor de **consultas**de **conexiones** &gt; de **opciones** *  &gt; de **herramientas** &gt; .
* Mediante programación, `servertimeout` establezca la propiedad de `System.TimeSpan`solicitud de cliente (un valor de tipo , hasta una hora).

Notas sobre los tiempos de espera:

* En el lado del cliente, el tiempo de espera se aplica desde la solicitud que se crea hasta el momento en que la respuesta comienza a llegar al cliente. El tiempo que se tarda en leer la carga en el cliente no se trata como parte del tiempo de espera (porque depende de la rapidez con la que el autor de la llamada extraiga los datos de la secuencia).
* También en el lado del cliente, el valor de tiempo de espera real utilizado es ligeramente superior al valor de tiempo de espera del servidor solicitado por el usuario. Esto es para permitir latencias de red.
* En el lado del servicio, no todos los operadores de consulta respetan el valor de tiempo de espera.
   Poco a poco vamos agregando ese apoyo.
* Para que Kusto utilice automáticamente el tiempo de `norequesttimeout` espera `true`de solicitud máximo permitido, establezca la propiedad de solicitud de cliente en .

<!--
  Request timeout can also be set using a set statement, but we don't mention
  it here as it should not be used in production scenarios.
-->

## <a name="limit-on-query-cpu-resource-usage"></a>Límite en el uso de recursos de CPU de consulta

De forma predeterminada, Kusto permite que las consultas en ejecución usen tantos recursos de CPU como el clúster tiene, intentando hacer un round-robin justo entre las consultas si se está ejecutando más de uno. En muchos casos, esto produce el mejor rendimiento para las consultas ad hoc.
En otros casos, es posible que se desee limitar los recursos de CPU asignados para una consulta determinada. Por ejemplo, si uno está ejecutando un "trabajo de fondo", uno podría tolerar latencias más altas para dar alta prioridad a las consultas ad hoc simultáneas.

Kusto admite la especificación de dos propiedades de solicitud de [cliente](../api/netfx/request-properties.md) al ejecutar una consulta, **query_fanout_threads_percent** y **query_fanout_nodes_percent**.
Ambos son enteros que tienen como valor predeterminado el valor máximo (100), pero que se pueden reducir para una consulta específica a algún valor. El primero controla el factor de fanout para el uso de subprocesos; cuando es 100% el clúster asignará todas las CPU en cada nodo (por ejemplo, en un clúster implementado en nodos de Azure D14, 16 CPU) a la consulta, cuando es 50% que la mitad de las CPU se usarán, etc. (los números se redondean a una CPU completa, por lo que es seguro establecerlo en 0.) El segundo controla cuántos nodos del clúster se utilizarán por operación de distribución de subconsulta y funciones de forma similar.

## <a name="limit-on-query-complexity"></a>Límite de complejidad de las consultas

Durante la ejecución de la consulta, el texto de la consulta se transforma en un árbol de operadores relacionales que representan la consulta.
En caso de que la profundidad del árbol supere un umbral interno (varios miles de niveles), la consulta se considera demasiado compleja para el procesamiento y se producirá un error con un código de error que indica que el árbol de operadores relacionales supera los límites.
En la mayoría de los casos, esto es causado por una consulta que contiene una larga lista de operadores binarios encadenados, por ejemplo:

```kusto
T 
| where Column == "value1" or 
        Column == "value2" or 
        .... or
        Column == "valueN"
```

Para este caso específico: se recomienda volver [`in()`](../query/inoperator.md) a escribir la consulta mediante operator. 

```kusto
T 
| where Column in ("value1", "value2".... "valueN")
```