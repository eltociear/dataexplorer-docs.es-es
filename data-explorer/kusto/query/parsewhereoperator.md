---
title: 'operador Parse-Where: Azure Explorador de datos'
description: En este artículo se describe el operador Parse-WHERE de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/12/2020
ms.openlocfilehash: 646ec00531d528efd51b4a168fde3de660a85ced
ms.sourcegitcommit: 733bde4c6bc422c64752af338b29cd55a5af1f88
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83271101"
---
# <a name="parse-where-operator"></a>Operador parse-where

Evalúa una expresión de cadena y analiza su valor en una o más columnas calculadas. El resultado es solo las cadenas analizadas correctamente.
Vea [operador Parse](parseoperator.md), que genera valores NULL para las cadenas analizadas incorrectamente.

```kusto
T | parse-where Text with "ActivityName=" name ", ActivityType=" type
```

**Sintaxis**

*T* `| parse-where` [ `kind=regex` [ `flags=regex_flags` ] | `simple` ] *expresión* `with` `*` (*StringConstant* *columnName* [ `:` *ColumnType*]) `*` ...

**Argumentos**

* *T*: la tabla de entrada.

* *tipo*: 

    * *simple* (valor predeterminado): StringConstant es un valor de cadena normal y la coincidencia es estricta. Todos los delimitadores de cadena deben aparecer en la cadena analizada y todas las columnas extendidas deben coincidir con los tipos necesarios.
        
    * *Regex*: StringConstant puede ser una expresión regular y la coincidencia es estricta. Todos los delimitadores de cadena deben aparecer en la cadena analizada y todas las columnas extendidas deben coincidir con los tipos necesarios. Los delimitadores de cadena pueden ser regex para este modo.
    
    * *marcas*: marcas que se van a usar en el modo regex: `U` (no expansivo), `m` (modo de varias líneas), `s` (coincidir con nueva línea `\n` ), `i` (no distingue mayúsculas de minúsculas), se pueden encontrar más marcas en [marcas RE2](re2.md).
        
* *Expresión*: expresión que se evalúa como una cadena.

* *ColumnName:* Nombre de una columna asignada a un valor que se ha sacado de la expresión de cadena. 
  
* *ColumnType:* debe ser un tipo escalar opcional que indica el tipo al que se va a convertir el valor. El valor predeterminado es tipo de cadena.

**Devuelve**

La tabla de entrada, que se extiende según la lista de columnas que se proporcionan al operador.

> [!Note] 
> Solo las cadenas analizadas correctamente estarán en la salida. Se filtrarán las cadenas que no coincidan con el patrón.

**Sugerencias**

* `parse-where`analiza las cadenas de la misma manera que el [análisis](parseoperator.md)y filtra las cadenas que no se analizaron correctamente.

* Use [Project](projectoperator.md) si también desea quitar o cambiar el nombre de algunas columnas.

* Use * en el patrón para omitir los valores no utilizados. Este valor no se puede usar después de la columna de cadena.

* El modelo de análisis puede empezar por *columnName*, además de *StringConstant*. 

* Si la *expresión* analizada no es de tipo cadena, se convertirá en una cadena de tipo.

* Si se utiliza el modo regex, puede agregar marcas regex para controlar toda la expresión regular utilizada en el análisis.

* En el modo regex, el análisis traducirá el patrón a regex y usará la [Sintaxis RE2](re2.md) para realizar la búsqueda de coincidencias con los grupos capturados numerados que se administran internamente.
  
  Por ejemplo, esta instrucción de análisis:
  
    ```kusto
    parse-where kind=regex Col with * <regex1> var1:string <regex2> var2:long
    ```

    El Regex que va a generar el análisis internamente es `.*?<regex1>(.*?)<regex2>(\-\d+)` .
        
    - `*`se ha traducido a `.*?` .
        
    - `string`se ha traducido a `.*?` .
        
    - `long`se ha traducido a `\-\d+` .

## <a name="examples"></a>Ejemplos

El `parse-where` operador proporciona una manera simplificada de `extend` una tabla mediante el uso `extract` de varias aplicaciones en la misma `string` expresión. Esto es muy útil cuando la tabla tiene una `string` columna que contiene varios valores que desea dividir en columnas individuales. Por ejemplo, puede dividir una columna generada por una instrucción Trace de desarrollador (" `printf` "/" `Console.WriteLine` ").

### <a name="using-parse"></a>Usar `parse`

En el ejemplo siguiente, la columna `EventText` de la tabla `Traces` contiene cadenas del formulario `Event: NotifySliceRelease (resourceName={0}, totalSlices= {1}, sliceNumber={2}, lockTime={3}, releaseTime={4}, previousLockTime={5})` . La operación siguiente extenderá la tabla con seis columnas: `resourceName` , `totalSlices` , `sliceNumber` , `lockTime ` , `releaseTime` , `previouLockTime` , `Month` y `Day` . 

Algunas de las cadenas no tienen una coincidencia completa.

Con `parse` , las columnas calculadas tendrán valores NULL.

<!-- csl: https://help.kusto.windows.net/Samples -->
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
| project resourceName ,totalSlices , sliceNumber , lockTime , releaseTime , previousLockTime
```

|resourceName|totalSlices|sliceNumber|lockTime|releaseTime|previousLockTime|
|---|---|---|---|---|---|
|||||||
|||||||
|||||||
|PipelineScheduler|27|20|02/17/2016 08:40:01|2016-02-17 08:40:01.0000000|2016-02-17 08:39:01.0000000|
|PipelineScheduler|27|22|02/17/2016 08:41:01|2016-02-17 08:41:00.0000000|2016-02-17 08:40:01.0000000|

### <a name="using-parse-where"></a>Usar `parse-where` 

El uso de ' Parse-Where ' filtrará las cadenas analizadas incorrectamente del resultado.

<!-- csl: https://help.kusto.windows.net/Samples -->
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
| parse-where EventText with * "resourceName=" resourceName ", totalSlices=" totalSlices:long * "sliceNumber=" sliceNumber:long * "lockTime=" lockTime ", releaseTime=" releaseTime:date "," * "previousLockTime=" previousLockTime:date ")" *  
| project resourceName ,totalSlices , sliceNumber , lockTime , releaseTime , previousLockTime
```

|resourceName|totalSlices|sliceNumber|lockTime|releaseTime|previousLockTime|
|---|---|---|---|---|---|
|PipelineScheduler|27|20|02/17/2016 08:40:01|2016-02-17 08:40:01.0000000|2016-02-17 08:39:01.0000000|
|PipelineScheduler|27|22|02/17/2016 08:41:01|2016-02-17 08:41:00.0000000|2016-02-17 08:40:01.0000000|


### <a name="regex-mode-using-regex-flags"></a>Modo regex mediante marcas Regex

Para obtener resourceName y totalSlices, use la siguiente consulta:

<!-- csl: https://help.kusto.windows.net/Samples -->
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

### <a name="parse-where-with-case-insensitive-regex-flag"></a>`parse-where`con la marca regex sin distinción entre mayúsculas y minúsculas

En la consulta anterior, el modo predeterminado distinguió entre mayúsculas y minúsculas, por lo que las cadenas se analizaron correctamente. No se obtuvo ningún resultado.

Para obtener el resultado necesario, ejecute `parse-where` con una marca regex sin distinción entre mayúsculas y minúsculas ( `i` ).

Solo se analizarán correctamente tres cadenas, por lo que el resultado es de tres registros (algunos totalSlices contienen enteros no válidos).

<!-- csl: https://help.kusto.windows.net/Samples -->
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
