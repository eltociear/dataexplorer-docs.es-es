---
title: Complemento de R (versión preliminar) - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe el complemento R (versión preliminar) en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: d815a75b241f7779a5f4ee9cae626c38ed54f9f4
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765995"
---
# <a name="r-plugin-preview"></a>Plugin R (versión preliminar)

::: zone pivot="azuredataexplorer"

El complemento R ejecuta una función definida por el usuario (UDF) mediante un script de R. El script de R obtiene datos tabulares como entrada y se espera que produzca resultados tabulares.
El tiempo de ejecución del complemento está alojado en un [entorno sandbox,](../concepts/sandboxes.md)un entorno aislado y seguro que se ejecuta en los nodos del clúster.

### <a name="syntax"></a>Sintaxis

*T* `|` `=` `single` `r(` *output_schema* `,` *script* `,` [`hint.distribution` ( | )] output_schema script [ script_parameters ] *script_parameters* `evaluate` `per_node``)`


### <a name="arguments"></a>Argumentos

* *output_schema*: `type` Un literal que define el esquema de salida de los datos tabulares, devueltos por el código R.
    * El formato `typeof(`es: *ColumnName* `:` *ColumnType* [, ...] `)`, por `typeof(col1:string, col2:long)`ejemplo: .
    * Para ampliar el esquema de entrada, utilice la sintaxis siguiente: `typeof(*, col1:string, col2:long)`.
* *script*: `string` Un literal que es el script de R válido que se va a ejecutar.
* *script_parameters*: `dynamic` Un literal opcional que es una bolsa de propiedades de pares `kargs` nombre/valor que se pasará al script de R como diccionario reservado (consulte [Variables de R reservadas](#reserved-r-variables)).
* *hint.distribution*: Una sugerencia opcional para que la ejecución del complemento se distribuya entre varios nodos del clúster.
   Valor predeterminado: `single`.
    * `single`: una única instancia del script se ejecutará en todos los datos de consulta.
    * `per_node`: si se distribuye la consulta antes de que se distribuya el bloque R, se ejecutará una instancia del script en cada nodo sobre los datos que contiene.


### <a name="reserved-r-variables"></a>Variables R reservadas

Las siguientes variables están reservadas para la interacción entre el lenguaje de consulta de Kusto y el código R:

* `df`: los datos tabulares `T` de entrada (los valores de arriba), como un DataFrame R.
* `kargs`: el valor del *argumento script_parameters,* como un diccionario R.
* `result`: un DataFrame de R creado por el script de R, cuyo valor se convierte en los datos tabulares que se envían a cualquier operador de consulta Kusto que siga al complemento.

### <a name="onboarding"></a>Incorporación


* El plugin está deshabilitado de forma predeterminada.
    * *¿Está interesado en habilitar el complemento en el clúster?*
        
        * En Azure Portal, en el clúster de Azure Data Explorer, seleccione Nueva solicitud de **soporte técnico** en el menú de la izquierda.
        * Deshabilitar el plugin también requiere abrir un ticket de soporte.

### <a name="notes-and-limitations"></a>Notas y limitaciones

* La imagen de entorno limitado de R se basa en *R 3.4.4 para Windows*e incluye paquetes del paquete R Essentials de [Anaconda.](https://docs.anaconda.com/anaconda/packages/r-language-pkg-docs/)
* El entorno limitado de R limita el acceso a la red, por lo tanto, el código de R no puede instalar dinámicamente paquetes adicionales que no se incluyen en la imagen. Abra una **nueva solicitud de soporte técnico** en Azure Portal si necesita paquetes específicos.


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

:::image type="content" source="images/samples/sine-demo.png" alt-text="Demostración sinusoito":::

### <a name="performance-tips"></a>Consejos de rendimiento

* Reduzca el conjunto de datos de entrada del plugin a la cantidad mínima requerida (columnas/filas).
    * Utilice filtros en el conjunto de datos de origen, cuando sea posible, mediante el lenguaje de consulta Kusto.
    * Para realizar un cálculo en un subconjunto de las columnas de origen, proyecte solo las columnas antes de invocar el complemento.
* Utilícelo `hint.distribution = per_node` siempre que la lógica del script sea distribuible.
    * También puede utilizar el operador de [partición](partitionoperator.md) para particionar el conjunto de datos de entrada.
* Siempre que sea posible, utilice el lenguaje de consulta Kusto para implementar la lógica del script de R.

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

* Para evitar conflictos entre delimitadores de cadena de Kusto y los`'`de R, se recomienda usar caracteres de comillas simples`"`( ) para literales de cadena de Kusto en consultas de Kusto y caracteres de comillas dobles ( ) para literales de cadena R en scripts de R.
* Use el [operador externaldata](externaldata-operator.md) para obtener el contenido de un script que ha almacenado en una ubicación externa, como Azure Blob Storage, un repositorio GitHub público, etc.
  
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

Esto no se admite en Azure Monitor

::: zone-end

