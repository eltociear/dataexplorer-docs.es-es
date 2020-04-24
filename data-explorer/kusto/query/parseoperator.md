---
title: 'operador de análisis: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe el operador de análisis en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: beb51ac80810e5d14013e9b3571d759bb7b09125
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81511516"
---
# <a name="parse-operator"></a>Operador parse

evalúa una expresión de cadena y analiza su valor en una o más columnas calculadas. Para las cadenas analizadas sin éxito, las columnas calculadas tendrán valores NULL.
Consulte el operador [de análisis donde](parsewhereoperator.md) filtra las cadenas analizadas sin éxito.

```kusto
T | parse Text with "ActivityName=" name ", ActivityType=" type
```

**Sintaxis**

*T* `| parse` `kind=regex` [`flags=regex_flags`[`simple` ] ? | `*` `*` *Expression* `with` `:` *ColumnType**StringConstant* ] Expresión ( StringConstant ColumnName [ ColumnType ]) ... *ColumnName* `relaxed`

**Argumentos**

* *T*: La tabla de entrada.
* Tipo: 

    * simple (valor predeterminado): StringConstant es un valor de cadena normal y la coincidencia es estricta, lo que significa que todos los delimitadores de cadena deben aparecer en la cadena analizada y todas las columnas extendidas deben coincidir con los tipos necesarios.
        
    * regex : StringConstant puede ser una expresión regular y la coincidencia es estricta, lo que significa que todos los delimitadores de cadena (que pueden ser una expresión regex para este modo) deben aparecer en la cadena analizada y todas las columnas extendidas deben coincidir con los tipos necesarios.
    
    * banderas : Indicadores que se `U` utilizarán en `m` el modo regex como `s` (Ungreedy), (modo multilínea), (coincide con la nueva línea), `\n` `i` (sin distinción entre mayúsculas y minúsculas) y más en los [indicadores RE2](re2.md).
        
    * relaxed : StringConstant es un valor de cadena normal y la coincidencia se relaja, lo que significa que todos los delimitadores de cadena deben aparecer en la cadena analizada, pero las columnas extendidas pueden coincidir con los tipos necesarios parcialmente (las columnas extendidas que no coinciden con los tipos necesarios obtendrán el valor null).

* *Expresión*: expresión que se evalúa como una cadena.

* *ColumnName:* El nombre de una columna a la que se va a asignar un valor (tomado de la expresión de cadena). 
  
* *ColumnType:* debe ser Tipo escalar opcional que indica el tipo en el que se va a convertir el valor (de forma predeterminada es tipo de cadena).

**Devuelve**

La tabla de entrada, extendida según la lista de columnas que se proporcionan al operador.

**Sugerencias**

* Utilícelo [`project`](projectoperator.md) si también desea quitar o cambiar el nombre de algunas columnas.

* Utilice * en el patrón para omitir los valores basura (no se puede utilizar después de la columna de cadena)

* El patrón de análisis puede comenzar con *ColumnName* y no solo con *StringConstant*. 

* Si la *expresión* analizada no es de tipo string , se convertirá en cadena de tipo.

* Si se utiliza el modo regex, hay una opción para agregar indicadores regex para controlar todo el regex utilizado en el análisis.

* En el modo regex, el análisis traducirá el patrón a una regex y utilizará la [sintaxis RE2](re2.md) para hacer la coincidencia usando los grupos capturados numerados que se manejan internamente.
  Así, por ejemplo, esta instrucción de análisis:
  
    ```kusto
    parse kind=regex Col with * <regex1> var1:string <regex2> var2:long
    ```

    El regex que generará el análisis `.*?<regex1>(.*?)<regex2>(\-\d+)`internamente es .
        
    - `*`fue traducido `.*?`a .
        
    - `string`fue traducido `.*?`a .
        
    - `long`fue traducido `\-\d+`a .

**Ejemplos**

El `parse` operador proporciona una `extend` forma simplificada `extract` de una `string` tabla mediante el uso de varias aplicaciones en la misma expresión.
Esto es más útil cuando `string` la tabla tiene una columna que contiene varios valores que desea dividir en`printf`columnas individuales, como una columna producida por una instrucción de seguimiento de desarrollador (" "/"`Console.WriteLine`").

En el ejemplo siguiente, `EventText` supongamos `Traces` que la `Event: NotifySliceRelease (resourceName={0}, totalSlices= {1}, sliceNumber={2}, lockTime={3}, releaseTime={4}, previousLockTime={5})`columna de tabla contiene cadenas del formulario .
La siguiente operación extenderá la `resourceName` tabla `totalSlices` `sliceNumber`con `lockTime ` `releaseTime`6 `previouLockTime` `Month` columnas: , , , , , , , , y `Day`. 


```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=23, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=02/17/2016 08:40:00, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces  
| parse EventText with * "resourceName=" resourceName ", totalSlices=" totalSlices:long * "sliceNumber=" sliceNumber:long * "lockTime=" lockTime ", releaseTime=" releaseTime:date "," * "previousLockTime=" previouLockTime:date ")" *  
| project resourceName ,totalSlices , sliceNumber , lockTime , releaseTime , previouLockTime
```

|resourceName|totalSlices|sliceNumber|lockTime|releaseTime|previouLockTime|
|---|---|---|---|---|---|
|PipelineScheduler|27|15|02/17/2016 08:40:00|2016-02-17 08:40:00.0000000|2016-02-17 08:39:00.0000000|
|PipelineScheduler|27|23|02/17/2016 08:40:01|2016-02-17 08:40:01.0000000|2016-02-17 08:39:01.0000000|
|PipelineScheduler|27|20|02/17/2016 08:40:01|2016-02-17 08:40:01.0000000|2016-02-17 08:39:01.0000000|
|PipelineScheduler|27|16|02/17/2016 08:41:00|2016-02-17 08:41:00.0000000|2016-02-17 08:40:00.0000000|
|PipelineScheduler|27|22|02/17/2016 08:41:01|2016-02-17 08:41:00.0000000|2016-02-17 08:40:01.0000000|

para el modo regex:

```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=23, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=02/17/2016 08:40:00, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces  
| parse kind = regex EventText with "(.*?)[a-zA-Z]*=" resourceName @", totalSlices=\s*\d+\s*.*?sliceNumber=" sliceNumber:long  ".*?(previous)?lockTime=" lockTime ".*?releaseTime=" releaseTime ".*?previousLockTime=" previouLockTime:date "\\)"  
| project resourceName , sliceNumber , lockTime , releaseTime , previouLockTime
```

|resourceName|sliceNumber|lockTime|releaseTime|previouLockTime|
|---|---|---|---|---|
|PipelineScheduler|15|02/17/2016 08:40:00, |02/17/2016 08:40:00, |2016-02-17 08:39:00.0000000|
|PipelineScheduler|23|02/17/2016 08:40:01, |02/17/2016 08:40:01, |2016-02-17 08:39:01.0000000|
|PipelineScheduler|20|02/17/2016 08:40:01, |02/17/2016 08:40:01, |2016-02-17 08:39:01.0000000|
|PipelineScheduler|16|02/17/2016 08:41:00, |02/17/2016 08:41:00, |2016-02-17 08:40:00.0000000|
|PipelineScheduler|22|02/17/2016 08:41:01, |02/17/2016 08:41:00, |2016-02-17 08:40:01.0000000|


para el modo regex utilizando indicadores regex:

si estamos interesados en obtener el resourceName solamente y usamos esta consulta:

```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=23, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=02/17/2016 08:40:00, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces
| parse kind = regex  EventText with * "resourceName=" resourceName ',' *
| project resourceName
```

|resourceName|
|---|
|PipelineScheduler, totalSlices-27, sliceNumber-23, lockTime-02/17/2016 08:40:01, releaseTime-02/17/2016 08:40:01|
|PipelineScheduler, totalSlices-27, sliceNumber-15, lockTime-02/17/2016 08:40:00, releaseTime-02/17/2016 08:40:00|
|PipelineScheduler, totalSlices-27, sliceNumber-20, lockTime-02/17/2016 08:40:01, releaseTime-02/17/2016 08:40:01|
|PipelineScheduler, totalSlices-27, sliceNumber-22, lockTime-02/17/2016 08:41:01, releaseTime-02/17/2016 08:41:00|
|PipelineScheduler, totalSlices-27, sliceNumber-16, lockTime-02/17/2016 08:41:00, releaseTime-02/17/2016 08:41:00|

Pero no obtenemos los resultados esperados ya que el modo predeterminado es codicioso.
o incluso si tuviéramos pocos registros donde el resourceName aparece a veces en mayúsculas, por lo que podemos obtener valores NULL para algunos valores.
para obtener el resultado deseado, podemos ejecutar este con los`U`indicadores regex`i`no codiciosos ( ) y deshabilitar los indicadores regex que distinguen mayúsculas de minúsculas ( ):

```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=23, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=02/17/2016 08:40:00, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=20, lockTime=02/17/2016 08:40:01, releaseTime=02/17/2016 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces
| parse kind = regex flags = Ui EventText with * "RESOURCENAME=" resourceName ',' *
| project resourceName
```

|resourceName|
|---|
|PipelineScheduler|
|PipelineScheduler|
|PipelineScheduler|
|PipelineScheduler|
|PipelineScheduler|


Si la cadena analizada tiene líneas nuevas, debe utilizar la marca `s` para analizar el texto como se esperaba:

```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler\ntotalSlices=27\nsliceNumber=23\nlockTime=02/17/2016 08:40:01\nreleaseTime=02/17/2016 08:40:01\npreviousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler\ntotalSlices=27\nsliceNumber=15\nlockTime=02/17/2016 08:40:00\nreleaseTime=02/17/2016 08:40:00\npreviousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler\ntotalSlices=27\nsliceNumber=20\nlockTime=02/17/2016 08:40:01\nreleaseTime=02/17/2016 08:40:01\npreviousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler\ntotalSlices=27\nsliceNumber=22\nlockTime=02/17/2016 08:41:01\nreleaseTime=02/17/2016 08:41:00\npreviousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler\ntotalSlices=27\nsliceNumber=16\nlockTime=02/17/2016 08:41:00\nreleaseTime=02/17/2016 08:41:00\npreviousLockTime=02/17/2016 08:40:00)"
];
Traces
| parse kind=regex flags=s EventText with * "resourceName=" resourceName:string "(.*?)totalSlices=" totalSlices:long "(.*?)lockTime=" lockTime:datetime "(.*?)releaseTime=" releaseTime:datetime "(.*?)previousLockTime=" previousLockTime:datetime "\\)" 
| project-away EventText
```

|resourceName|totalSlices|lockTime|releaseTime|anteriorLockTime|
|---|---|---|---|---|
|PipelineScheduler<br>|27|2016-02-17 08:40:00.0000000|2016-02-17 08:40:00.0000000|2016-02-17 08:39:00.0000000|
|PipelineScheduler<br>|27|2016-02-17 08:40:01.0000000|2016-02-17 08:40:01.0000000|2016-02-17 08:39:01.0000000|
|PipelineScheduler<br>|27|2016-02-17 08:40:01.0000000|2016-02-17 08:40:01.0000000|2016-02-17 08:39:01.0000000|
|PipelineScheduler<br>|27|2016-02-17 08:41:00.0000000|2016-02-17 08:41:00.0000000|2016-02-17 08:40:00.0000000|
|PipelineScheduler<br>|27|2016-02-17 08:41:01.0000000|2016-02-17 08:41:00.0000000|2016-02-17 08:40:01.0000000|


en este ejemplo para el modo relajado: totalSlices columna extendida es necesario ser de tipo long, pero en la cadena analizada tiene el valor nonValidLongValue.
ReleaseTime columna extendida tiene el mismo problema, el valor nonValidDateTime no se puede analizar como datetime.
en este caso, estas dos columnas extendidas obtendrán el valor null, mientras que las otras (como sliceNumber) todavía obtienen los valores correctos.

usar el tipo para la misma consulta a continuación da null para todas las columnas extendidas porque es estricto en columnas extendidas (esa es la diferencia entre el modo relajado y simple, en modo relajado, las columnas extendidas se pueden hacer coincidir parcialmente).

```kusto
let Traces = datatable(EventText:string)
[
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=23, lockTime=02/17/2016 08:40:01, releaseTime=nonValidDateTime 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=15, lockTime=02/17/2016 08:40:00, releaseTime=nonValidDateTime, previousLockTime=02/17/2016 08:39:00)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=nonValidLongValue, sliceNumber=20, lockTime=02/17/2016 08:40:01, releaseTime=nonValidDateTime 08:40:01, previousLockTime=02/17/2016 08:39:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=27, sliceNumber=22, lockTime=02/17/2016 08:41:01, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:01)",
"Event: NotifySliceRelease (resourceName=PipelineScheduler, totalSlices=nonValidLongValue, sliceNumber=16, lockTime=02/17/2016 08:41:00, releaseTime=02/17/2016 08:41:00, previousLockTime=02/17/2016 08:40:00)"
];
Traces
| parse kind=relaxed EventText with * "resourceName=" resourceName ", totalSlices=" totalSlices:long * "sliceNumber=" sliceNumber:long * "lockTime=" lockTime ", releaseTime=" releaseTime:date "," * "previousLockTime=" previouLockTime:date ")" *
| project-away EventText
```

|resourceName|totalSlices|sliceNumber|lockTime|releaseTime|previouLockTime|
|---|---|---|---|---|---|
|PipelineScheduler|27|15|02/17/2016 08:40:00||2016-02-17 08:39:00.0000000|
|PipelineScheduler|27|23|02/17/2016 08:40:01||2016-02-17 08:39:01.0000000|
|PipelineScheduler||20|02/17/2016 08:40:01||2016-02-17 08:39:01.0000000|
|PipelineScheduler||16|02/17/2016 08:41:00|2016-02-17 08:41:00.0000000|2016-02-17 08:40:00.0000000|
|PipelineScheduler|27|22|02/17/2016 08:41:01|2016-02-17 08:41:00.0000000|2016-02-17 08:40:01.0000000|