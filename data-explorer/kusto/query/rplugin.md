---
title: 'Complemento de R (versión preliminar): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe R plugin (versión preliminar) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 1014b9090fef60816c4bbc0a7fd9b2fdc4c22801
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737800"
---
# <a name="r-plugin-preview"></a>Complemento de R (versión preliminar)

::: zone pivot="azuredataexplorer"

El complemento de R ejecuta una función definida por el usuario (UDF) mediante un script de R. El script de R obtiene los datos tabulares como entrada y se espera que genere resultados tabulares.
El tiempo de ejecución del complemento se hospeda en un [espacio aislado](../concepts/sandboxes.md), un entorno aislado y seguro que se ejecuta en los nodos del clúster.

### <a name="syntax"></a>Sintaxis

*T* `|` `per_node``,` *script_parameters* *output_schema* *script* [`hint.distribution` (`single`)] `r(`output_schema`,` script [script_parameters] `evaluate` `=`  | `)`


### <a name="arguments"></a>Argumentos

* *output_schema*: un `type` literal que define el esquema de salida de los datos tabulares, devueltos por el código de R.
    * El formato es: `typeof(` *columnName* `:` *ColumnType* [,...] `)`, por ejemplo: `typeof(col1:string, col2:long)`.
    * Para extender el esquema de entrada, use la sintaxis siguiente `typeof(*, col1:string, col2:long)`:.
* *script*: un `string` literal que es el script de R válido que se va a ejecutar.
* *script_parameters*: un literal `dynamic` opcional que es un contenedor de propiedades de pares de nombre y valor que se va a pasar al script de `kargs` r como diccionario reservado (consulte [variables de r reservadas](#reserved-r-variables)).
* *Hint. Distribution*: sugerencia opcional para que la ejecución del complemento se distribuya en varios nodos del clúster.
   Predeterminado: `single`.
    * `single`: Una instancia única del script se ejecutará en todos los datos de la consulta.
    * `per_node`: Si la consulta antes del bloque de R se distribuye, una instancia del script se ejecutará en cada nodo sobre los datos que contiene.


### <a name="reserved-r-variables"></a>Variables de R reservadas

Las siguientes variables están reservadas para la interacción entre el lenguaje de consulta de Kusto y el código de R:

* `df`: Los datos tabulares de entrada (los `T` valores de arriba), como una trama de datos de R.
* `kargs`: El valor del argumento *script_parameters* , como un diccionario de R.
* `result`: Una trama de datos de R creada por el script de R, cuyo valor se convierte en los datos tabulares que se envían a cualquier operador de consulta Kusto que siga el complemento.

### <a name="onboarding"></a>Incorporación


* El complemento está deshabilitado de forma predeterminada.
    * *¿Está interesado en habilitar el complemento en el clúster?*
        
        * En el Azure Portal, en el clúster de Azure Explorador de datos, seleccione **nueva solicitud de soporte técnico** en el menú de la izquierda.
        * Deshabilitar el complemento requiere también abrir una incidencia de soporte técnico.

### <a name="notes-and-limitations"></a>Notas y limitaciones

* La imagen de espacio aislado de R se basa en *r 3.4.4 para Windows*e incluye paquetes del [paquete de r Essentials de Anaconda](https://docs.anaconda.com/anaconda/packages/r-language-pkg-docs/).
* El espacio aislado de R limita el acceso a la red, por lo que el código de R no puede instalar dinámicamente paquetes adicionales que no se incluyen en la imagen. Abra una **nueva solicitud de soporte técnico** en el Azure portal si necesita paquetes específicos.


### <a name="examples"></a>Ejemplos

```kusto
range x from 1 to 360 step 1
| evaluate r(
//
typeof(*, fx:double),               //  Output schema: append a new fx column to original table 
//
'result <- df\n'                    //  The R decorated script
'n <- nrow(df)\n'
'g <- kargs$gain\n'
'f <- kargs$cycles\n'
'result$fx <- g * sin(df$x / n * 2 * pi * f)'
//
, pack('gain', 100, 'cycles', 4)    //  dictionary of parameters
)
| render linechart 
```

:::image type="content" source="images/plugin/sine-demo.png" alt-text="Demo sinusoidal" border="false":::

### <a name="performance-tips"></a>Consejos de rendimiento

* Reduzca el conjunto de datos de entrada del complemento a la cantidad mínima necesaria (columnas o filas).
    * Utilice filtros en el conjunto de datos de origen, siempre que sea posible, mediante el lenguaje de consulta Kusto.
    * Para realizar un cálculo en un subconjunto de las columnas de origen, solo tiene que proyectar esa columna antes de invocar el complemento.
* Use `hint.distribution = per_node` siempre que la lógica del script sea distribuible.
    * También puede usar el [operador Partition](partitionoperator.md) para particionar el conjunto de datos de entrada.
* Siempre que sea posible, use el lenguaje de consulta Kusto para implementar la lógica de su script de R.

    Por ejemplo:

    ```kusto    
    .show operations
    | where StartedOn > ago(1d) // Filtering out irrelevant records before invoking the plugin
    | project d_seconds = Duration / 1s // Projecting only a subset of the necessary columns
    | evaluate hint.distribution = per_node r( // Using per_node distribution, as the script's logic allows it
        typeof(*, d2:double),
        'result <- df\n'
        'result$d2 <- df$d_seconds\n' // Negative example: this logic should have been written using Kusto's query language
      )
    | summarize avg = avg(d2)
    ```

### <a name="usage-tips"></a>Consejos de uso

* Para evitar conflictos entre delimitadores de cadena de Kusto y R, se recomienda usar caracteres de comillas`'`simples () para literales de cadena de Kusto en consultas de Kusto y`"`caracteres de comillas dobles () para literales de cadena de r en scripts de r.
* Use el [operador externaldata](externaldata-operator.md) para obtener el contenido de un script que ha almacenado en una ubicación externa, como Azure BLOB Storage, un repositorio público de Github, etc.
  
  Por ejemplo:

    ```kusto    
    let script = 
        externaldata(script:string)
        [h'https://kustoscriptsamples.blob.core.windows.net/samples/R/sample_script.r']
        with(format = raw);
    range x from 1 to 360 step 1
    | evaluate r(
        typeof(*, fx:double),
        toscalar(script), 
        pack('gain', 100, 'cycles', 4))
    | render linechart 
    ```

---

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end

