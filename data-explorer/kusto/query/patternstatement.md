---
title: 'instrucción Pattern: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la instrucción Pattern en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 97fc8361feb3d8dddb7d0722ce741ef44bd4cec6
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737273"
---
# <a name="pattern-statement"></a>Pattern (instrucción)

::: zone pivot="azuredataexplorer"

Un **patrón** es una construcción de tipo "vista" con nombre que asigna tuplas de cadena predefinidas a cuerpos de función sin parámetros. Los patrones son únicos en dos aspectos:

* Los patrones se "invocan" mediante una sintaxis similar a las referencias de tabla de ámbito.
* Los patrones tienen un conjunto de valores de argumento controlado, de cierre cerrado que se pueden asignar y el proceso de asignación se realiza mediante Kusto. Esto significa que, si se declara un patrón pero no se define, Kusto identifica y marca como error todas las invocaciones al patrón, lo que permite "resolver" estos patrones mediante una aplicación de nivel intermedio.


## <a name="pattern-declaration"></a>Declaración de patrón
La instrucción Pattern se usa para declarar o definir un modelo.
Por ejemplo, la siguiente es una instrucción pattern que declara `app` como un patrón:

```kusto
declare pattern app;
```

Esta instrucción indica a Kusto `app` que es un patrón, pero no indica a Kusto cómo resolver el patrón. Como resultado, cualquier intento de invocar este patrón en la consulta producirá un error específico que enumerará todas las invocaciones. Por ejemplo:

```kusto
declare pattern app;
app("ApplicationX").StartEvents
| join kind=inner app("ApplicationX").StopEvents on CorrelationId
| count
```

Esta consulta generará un error de Kusto, que indica que no se pueden resolver las siguientes invocaciones de `app("ApplicationX")["StartEvents"]` patrón `app("ApplicationX")["StopEvents"]`: y.

**Sintaxis**

`declare``pattern` *PatternName*

## <a name="pattern-definition"></a>Definición de patrón

La instrucción Pattern también se puede utilizar para definir un patrón. En una definición de patrón, todas las posibles invocaciones del patrón se disponen explícitamente y se proporciona la expresión tabular correspondiente. Cuando Kusto ejecuta la consulta, reemplaza cada invocación de patrón con el cuerpo del patrón correspondiente. Por ejemplo:

```kusto
declare pattern app = (applicationId:string)[eventType:string]
{
    ("ApplicationX").["StopEvents"] = { database("AppX").Events | where EventType == "StopEvent" };
    ("ApplicationX").["StartEvents"] = { database(applicationId).Events | where EventType == eventType } ;
};
app("ApplicationX").StartEvents
| join kind=inner app("ApplicationX").StopEvents on CorrelationId
| count
```

La expresión que se proporciona para cada patrón coincidente es un nombre de tabla o una referencia a una [instrucción Let](letstatement.md).

**Sintaxis**

`declare``pattern` *PatternName*PatternName = *ArgName* ArgName `:` *ArgType* [`,` ..`(`.] `)` [`[` *PathName* Nombreruta `:` *PathArgType* PathArgType `]`]`{`
&nbsp;&nbsp;&nbsp;&nbsp;`(` *ArgValue1* [`,` *ArgValue2* ...] `)` [ `.[` * PathValue `]` ] `=` `{` *expresión* `(` *ArgValue1_2* *ArgValue2_2* [ArgValue1_2 [`,` ArgValue2_2.. &nbsp; .]`};` &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; `)` [ `.[` *PathValue_2* `{` *expression_2* ] `=` expression_2`};` .. &nbsp; . `]` &nbsp; &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp; ]        `}`

* *PatternName*: nombre de la palabra clave pattern. Solo se permite la sintaxis que define la palabra clave: para detectar todas las referencias de patrón con una palabra clave especificada.
* *ArgName*: nombre del argumento pattern. Los patrones permiten uno o varios nombres de argumento.
* *ArgType*: tipo del argumento Pattern (actualmente solo `string` se permite)
* *PathName*: nombre del argumento path. Los patrones permiten un nombre de ruta de acceso o cero.
* *PathType*: tipo del argumento path (actualmente solo `string` se permite)
* *ArgValue1*, *ArgValue2*,...: valores de los argumentos de patrón (actualmente `string` solo se permiten literales)
* *PathValue* : valor de la ruta de acceso del patrón `string` (actualmente solo se permiten literales)
* *expresión* *: expresión* tabular (por ejemplo, `Logs | where Timestamp > ago(1h)`) o expresión lambda que hace referencia a una función.

## <a name="pattern-invocation"></a>Invocación de patrón

La sintaxis de invocación de patrón es similar a la sintaxis de referencia de tabla de ámbito:

* *PatternName* `(` *ArgValue1* [`,` *ArgValue2* ...] `).` *PathValue*
* *PatternName* `(` *ArgValue1* [`,` *ArgValue2* ...] `).["` *PathValue*`"]`

## <a name="remarks"></a>Comentarios

**Escenario**

La instrucción pattern está diseñada para aplicaciones de nivel intermedio que aceptan consultas de usuario y, a continuación, envían estas consultas a Kusto. Estas aplicaciones suelen prefijar esas consultas de usuario con un modelo de esquema lógico (un conjunto de [instrucciones Let](letstatement.md), posiblemente subiendo por una [instrucción Restrict](restrictstatement.md)).
En algunos casos, estas aplicaciones necesitan una sintaxis que pueden permitir a los usuarios hacer referencia a entidades que no se conocen con antelación y que se definen en el esquema lógico que construyen (ya sea porque no se conocen con antelación, porque el número de entidades potenciales es demasiado grande para estar predefinido en el esquema lógico.

El patrón soluciona este escenario de la siguiente manera. La aplicación de nivel intermedio envía la consulta a Kusto con todos los patrones declarados, pero no definidos. A continuación, Kusto analiza la consulta y, si hay una o varias invocaciones de patrón, devuelve un error a la aplicación de nivel intermedio con todas las invocaciones indicadas explícitamente. A continuación, la aplicación de nivel intermedio puede resolver cada una de estas referencias y volver a ejecutar la consulta, con lo que se prefijará con la definición de patrón totalmente elaborada.

**Normalización**

Kusto normaliza automáticamente el patrón, por lo que a continuación se muestran todas las invocaciones del mismo patrón y se devuelve una sola vez:

```kusto
declare pattern app;
union
  app("ApplicationX").StartEvent,
  app('ApplicationX').StartEvent,
  app("ApplicationX").['StartEvent'],
  app("ApplicationX").["StartEvent"]
```

Esto también significa que uno no puede definirlos juntos, ya que se consideran iguales.

**Caracteres comodín**

Kusto no trata los caracteres comodín en un patrón de una manera especial. Por ejemplo, en la siguiente consulta:

```kusto
declare pattern app;
union app("ApplicationX").*
| count
```

Kusto informará de una única invocación de `app("ApplicationX").["*"]`patrón que falta:.

## <a name="examples"></a>Ejemplos

Las consultas superan una única invocación de patrón:

```kusto
declare pattern A
{
    // ...
};
union (A('a1').Text), (A('a2').Text)
```

|Aplicación|SomeText|
|---|---|
|#1 de aplicación|Este es un texto gratis: 1|
|#1 de aplicación|Este es un texto gratis: 2|
|#1 de aplicación|Este es un texto gratis: 3|
|#1 de aplicación|Este es un texto gratis: 4|
|#1 de aplicación|Este es un texto gratis: 5|
|#2 de aplicación|Este es un texto gratis: 9|
|#2 de aplicación|Este es un texto gratis: 8|
|#2 de aplicación|Este es un texto gratis: 7|
|#2 de aplicación|Este es un texto gratis: 6|
|#2 de aplicación|Este es un texto gratis: 5|

```kusto
declare pattern App;
union (App('a1').Text), (App('a2').Text)
```

Error semántico:

     SEM0036: One or more pattern references were not declared. Detected pattern references: ["App('a1').['Text']","App('a2').['Text']"].

```kusto
declare pattern App;
declare pattern App = (applicationId:string)[scope:string]  
{
    ('a1').['Data']    = { range x from 1 to 5 step 1 | project App = "App #1", Data    = x };
    ('a1').['Metrics'] = { range x from 1 to 5 step 1 | project App = "App #1", Metrics = rand() };
    ('a2').['Data']    = { range x from 1 to 5 step 1 | project App = "App #2", Data    = 10 - x };
    ('a3').['Metrics'] = { range x from 1 to 5 step 1 | project App = "App #3", Metrics = rand() };
};
union (App('a2').Metrics), (App('a3').Metrics) 
```

Error semántico devuelto:

    SEM0036: One or more pattern references were not declared. Detected pattern references: ["App('a2').['Metrics']","App('a3').['Metrics']"].

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end
