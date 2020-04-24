---
title: operador de análisis- Azure Data Explorer ? Microsoft Docs
description: En este artículo se describe el operador de análisis en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/12/2020
ms.openlocfilehash: 0e44ab83242fc8aed0e46bdab1fa5a142992bfe5
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511414"
---
# <a name="parse-where-operator"></a>Operador parse-where

evalúa una expresión de cadena y analiza su valor en una o más columnas calculadas. El resultado son solo las cadenas analizadas correctamente.
Consulte el operador de [análisis](parseoperator.md) que genera valores NULL para cadenas analizadas sin éxito.

```kusto
T | parse-where Text with "ActivityName=" name ", ActivityType=" type
```

**Sintaxis**

*T* `| parse-where` `kind=regex` [`flags=regex_flags`[`simple`] ] *Expression* `with` Expresión `*` (*StringConstant* `:` *ColumnName* [ `*` *ColumnType*]) ...

**Argumentos**

* *T*: La tabla de entrada.

* Tipo: 

    * simple (valor predeterminado): StringConstant es un valor de cadena normal y la coincidencia es estricta, lo que significa que todos los delimitadores de cadena deben aparecer en la cadena analizada y todas las columnas extendidas deben coincidir con los tipos necesarios.
        
    * regex : StringConstant puede ser una expresión regular y la coincidencia es estricta, lo que significa que todos los delimitadores de cadena (que pueden ser una expresión regex para este modo) deben aparecer en la cadena analizada y todas las columnas extendidas deben coincidir con los tipos necesarios.
    
    * banderas : Indicadores que se `U` utilizarán en `m` el modo regex como `s` (Ungreedy), (modo multilínea), (coincide con la nueva línea), `\n` `i` (sin distinción entre mayúsculas y minúsculas) y más en los [indicadores RE2](re2.md).
        
* *Expresión*: expresión que se evalúa como una cadena.

* *ColumnName:* El nombre de una columna a la que se va a asignar un valor (tomado de la expresión de cadena). 
  
* *ColumnType:* debe ser Tipo escalar opcional que indica el tipo en el que se va a convertir el valor (de forma predeterminada es tipo de cadena).

**Devuelve**

La tabla de entrada, extendida según la lista de columnas que se proporcionan al operador.
Solo las cadenas analizadas correctamente estarán en la salida. las cadenas que no coincidan con el patrón se filtrarán.

**Sugerencias**

* `parse-where`analiza las cadenas de la misma manera que [analizan,](parseoperator.md) pero además filtra las cadenas que no se analizaron correctamente.

* Utilícelo [`project`](projectoperator.md) si también desea quitar o cambiar el nombre de algunas columnas.

* Utilice * en el patrón para omitir los valores basura (no se puede utilizar después de la columna de cadena)

* El patrón de análisis puede comenzar con *ColumnName* y no solo con *StringConstant*. 

* Si la *expresión* analizada no es de tipo string , se convertirá en cadena de tipo.

* Si se utiliza el modo regex, hay una opción para agregar indicadores regex para controlar todo el regex utilizado en el análisis.

* En el modo regex, el análisis traducirá el patrón a una regex y utilizará la [sintaxis RE2](re2.md) para hacer la coincidencia usando los grupos capturados numerados que se manejan internamente.
  
  Así, por ejemplo, esta instrucción de análisis:
  
    ```kusto
    parse-where kind=regex Col with * <regex1> var1:string <regex2> var2:long
    ```

    El regex que generará el análisis `.*?<regex1>(.*?)<regex2>(\-\d+)`internamente es .
        
    - `*`fue traducido `.*?`a .
        
    - `string`fue traducido `.*?`a .
        
    - `long`fue traducido `\-\d+`a .

**Ejemplos**

El `parse-where` operador proporciona una `extend` forma simplificada `extract` de una `string` tabla mediante el uso de varias aplicaciones en la misma expresión.
Esto es más útil cuando `string` la tabla tiene una columna que contiene varios valores que desea dividir en`printf`columnas individuales, como una columna producida por una instrucción de seguimiento de desarrollador (" "/"`Console.WriteLine`").

En el ejemplo siguiente, `EventText` supongamos `Traces` que la `Event: NotifySliceRelease (resourceName={0}, totalSlices= {1}, sliceNumber={2}, lockTime={3}, releaseTime={4}, previousLockTime={5})`columna de tabla contiene cadenas del formulario .
La siguiente operación extenderá la `resourceName` tabla `totalSlices` `sliceNumber`con `lockTime ` `releaseTime`6 `previouLockTime` `Month` columnas: , , , , , , , , y `Day`. 

Aquí, pocas cuerdas no tienen una coincidencia completa.
Usando `parse`, las columnas calculadas tendrán valores NULL:

```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=invalid_number, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=invalid_datetime, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=invalid_number, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces  
| parse EventText with * "resourceName=" resourceName ", totalSlices=" totalSlices:long * "sliceNumber=" sliceNumber:long * "lockTime=" lockTime ", releaseTime=" releaseTime:date "," * "previousLockTime=" previouLockTime:date ")" *  
| project resourceName ,totalSlices , sliceNumber , lockTime , releaseTime , previouLockTime
```

|resourceName|totalSlices|sliceNumber|lockTime|releaseTime|previouLockTime|
|---|---|---|---|---|---|
|||||||
|||||||
|||||||
|PipelineScheduler|27|20|02/17/2016 08:40:01|2016-02-17 08:40:01.0000000|2016-02-17 08:39:01.0000000|
|PipelineScheduler|27|22|02/17/2016 08:41:01|2016-02-17 08:41:00.0000000|2016-02-17 08:40:01.0000000|


El `parse-where` uso filtrará las cadenas analizadas sin éxito del resultado:

```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=invalid_number, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=invalid_datetime, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=invalid_number, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces  
| parse-where EventText with * "resourceName=" resourceName ", totalSlices=" totalSlices:long * "sliceNumber=" sliceNumber:long * "lockTime=" lockTime ", releaseTime=" releaseTime:date "," * "previousLockTime=" previouLockTime:date ")" *  
| project resourceName ,totalSlices , sliceNumber , lockTime , releaseTime , previouLockTime
```

|resourceName|totalSlices|sliceNumber|lockTime|releaseTime|previouLockTime|
|---|---|---|---|---|---|
|PipelineScheduler|27|20|02/17/2016 08:40:01|2016-02-17 08:40:01.0000000|2016-02-17 08:39:01.0000000|
|PipelineScheduler|27|22|02/17/2016 08:41:01|2016-02-17 08:41:00.0000000|2016-02-17 08:40:01.0000000|


para el modo regex utilizando indicadores regex:

si estamos interesados en obtener el resouceName y totalSlices y usamos esta consulta:

```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=non_valid_integer, sliceNumber=11, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=02/17/2016 08:40:00, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=non_valid_integer, sliceNumber=44, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces
| parse-where kind = regex EventText with * "RESOURCENAME=" resourceName "," * "totalSlices=" totalSlices:long "," *
| project resourceName, totalSlices
```

En la consulta anterior no obtenemos ningún resultado ya que el modo predeterminado distingue mayúsculas de minúsculas, por lo que ninguna de las cadenas se analizó correctamente.

con el fin de obtener el `parse-where` resultado requerido, podemos`i`ejecutar el indicador regex con mayúsculas y minúsculas ( ).
Tenga en cuenta que solo 3 cadenas se analizarán correctamente y, por lo tanto, el resultado es de 3 registros (algunos totalSlices contienen enteros no válidos): 

```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=non_valid_integer, sliceNumber=11, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=02/17/2016 08:40:00, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=non_valid_integer, sliceNumber=44, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces
| parse-where kind = regex flags=i EventText with * "RESOURCENAME=" resourceName "," * "totalSlices=" totalSlices:long "," *
| project resourceName, totalSlices
```

|resourceName|totalSlices|
|---|---|
|PipelineScheduler|27|
|PipelineScheduler|27|
|PipelineScheduler|27|


