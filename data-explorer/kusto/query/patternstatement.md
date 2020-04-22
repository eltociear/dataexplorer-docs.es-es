---
title: 'Declaración de patrón: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la instrucción de patrón en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: d7ad021fe7b458d54beeb24b908420324b45ad39
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765668"
---
# <a name="pattern-statement"></a>declaración de patrón

::: zone pivot="azuredataexplorer"

Un **patrón** es una construcción con nombre de vista que asigna tuplas de cadena predefinidas a cuerpos de función sin parámetros. Los patrones son únicos en dos aspectos:

* Los patrones se "invocan" mediante una sintaxis similar a las referencias de tabla con ámbito.
* Los patrones tienen un conjunto controlado, cerrado, de valores de argumento que se pueden asignar y Kusto realiza el proceso de asignación. Esto significa que si se declara un patrón pero no se define, Kusto identifica y marca como un error todas las invocaciones al patrón, lo que permite "resolver" estos patrones mediante una aplicación de nivel intermedio.


## <a name="pattern-declaration"></a>Declaración de patrón
La instrucción pattern se utiliza para declarar o definir un patrón.
Por ejemplo, la siguiente es una `app` instrucción pattern que declara ser un patrón:

```kusto
declare pattern app;
```

Esta declaración le dice `app` a Kusto que es un patrón, pero no le dice a Kusto cómo resolver el patrón. Como resultado, cualquier intento de invocar este patrón en la consulta dará lugar a un error específico que enumera todas estas invocaciones. Por ejemplo:

```kusto
declare pattern app;
app("ApplicationX").StartEvents
| join kind=inner app("ApplicationX").StopEvents on CorrelationId
| count
```

Esta consulta generará un error de Kusto, que indica que no `app("ApplicationX")["StartEvents"]` `app("ApplicationX")["StopEvents"]`se pueden resolver las siguientes invocaciones de patrón: y .

**Sintaxis**

`declare``pattern` *PatternName*

## <a name="pattern-definition"></a>Definición de patrón

La instrucción pattern también se puede utilizar para definir un patrón. En una definición de patrón, todas las invocaciones posibles del patrón se establecen explícitamente y se da la expresión tabular correspondiente. Cuando Kusto, a continuación, ejecuta la consulta, reemplaza cada invocación de patrón con el cuerpo del patrón correspondiente. Por ejemplo:

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

La expresión que se proporciona para cada patrón que se va a hacer coincidir es un nombre de tabla o una referencia a una [instrucción let](letstatement.md).

**Sintaxis**

`declare``pattern` *PatternName* = *ArgName* `:` *ArgType* `,` ArgName ArgType [ ... ]`(` `)` [`[` *PathName* `:` *PathArgType* `]`]`{`
&nbsp;&nbsp;&nbsp;&nbsp;`(` *ArgValue1* `,` [ *ArgValue2* ... ] `)` [Expresión `.[` `]` *PathValue `=` `{` *]* `};` &nbsp; &nbsp; &nbsp; `,` *ArgValue2_2* *ArgValue1_2* [ &nbsp; &nbsp; &nbsp; [ ArgValue2_2 ... ] &nbsp; &nbsp; `(` `)` [ `.[` *PathValue_2* `]` PathValue_2 `=` `{`] *expression_2* `};` expression_2 &nbsp; &nbsp; ... &nbsp; &nbsp; &nbsp;&nbsp;&nbsp;&nbsp; ]        `}`

* *PatternName*: Nombre de la palabra clave pattern. Sintaxis que define solo palabras clave: para detectar todas las referencias de patrón con una palabra clave especificada.
* *ArgName*: Nombre del argumento de patrón. Los patrones permiten uno o varios nombres de argumento.
* *ArgType*: Tipo del argumento `string` de patrón (actualmente solo se permite)
* *PathName*: nombre del argumento path. Los patrones permiten un nombre de ruta cero o uno.
* *PathType*: Tipo del argumento `string` path (actualmente solo se permite)
* *ArgValue1*, *ArgValue2*, ... - valores de `string` los argumentos de patrón (actualmente solo se permiten literales)
* *PathValue* - valor de la `string` ruta de acceso del patrón (actualmente solo se permiten literales)
* *expresión*: La *expresión* - una `Logs | where Timestamp > ago(1h)`expresión tabular (por ejemplo, ), o una expresión lambda que hace referencia a una función.

## <a name="pattern-invocation"></a>Invocación de patrones

La sintaxis de invocación de patrón es similar a la sintaxis de referencia de tabla con ámbito:

* *PatternName* `(` *ArgValue1* [`,` *ArgValue2* ...] `).` *PathValue*
* *PatternName* `(` *ArgValue1* [`,` *ArgValue2* ...] `).["` *PathValue*`"]`

## <a name="remarks"></a>Observaciones

**Escenario**

La instrucción pattern está diseñada para aplicaciones de nivel intermedio que aceptan consultas de usuario y, a continuación, envían estas consultas a Kusto. Estas aplicaciones a menudo anteponen esas consultas de usuario con un modelo de esquema lógico (un conjunto de [instrucciones let,](letstatement.md)posiblemente sufijadas por una [instrucción restrict).](restrictstatement.md)
En algunos casos, estas aplicaciones necesitan una sintaxis que pueden proporcionar a los usuarios a hacer referencia a entidades que no se pueden conocer con antelación y definir en el esquema lógico que construyen (ya sea porque no se conocen con antelación, porque el número de entidades potenciales es demasiado grande para ser predefinida en el esquema lógico.

El patrón resuelve este escenario de la siguiente manera. La aplicación de nivel intermedio envía la consulta a Kusto con todos los patrones declarados, pero no definidos. Kusto, a continuación, analiza la consulta y, si hay una o más invocaciones de patrón en ella, devuelve un error a la aplicación de nivel intermedio con todas estas invocaciones enumeradas explícitamente. A continuación, la aplicación de nivel intermedio puede resolver cada una de estas referencias y volver a ejecutar la consulta, esta vez prefijándola con la definición de patrón completamente elaborada.

**Normalizaciones**

Kusto normaliza automáticamente el patrón, por lo que, por ejemplo, las siguientes son todas invocaciones del mismo patrón, y se notifica una sola:

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

Kusto no trata los comodines en un patrón de ninguna manera especial. Por ejemplo, en la siguiente consulta:

```kusto
declare pattern app;
union app("ApplicationX").*
| count
```

Kusto informará de una sola `app("ApplicationX").["*"]`invocación de patrón que falta: .

## <a name="examples"></a>Ejemplos

Consultas sobre más de una sola invocación de patrón:

```kusto
declare pattern A
{
    // ...
};
union (A('a1').Text), (A('a2').Text)
```

|Aplicación|SomeText|
|---|---|
|#1 de la aplicación|Este es un texto libre: 1|
|#1 de la aplicación|Este es un texto libre: 2|
|#1 de la aplicación|Este es un texto libre: 3|
|#1 de la aplicación|Este es un texto libre: 4|
|#1 de la aplicación|Este es un texto libre: 5|
|#2 de la aplicación|Este es un texto libre: 9|
|#2 de la aplicación|Este es un texto libre: 8|
|#2 de la aplicación|Este es un texto libre: 7|
|#2 de la aplicación|Este es un texto libre: 6|
|#2 de la aplicación|Este es un texto libre: 5|

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

Esto no se admite en Azure Monitor

::: zone-end
