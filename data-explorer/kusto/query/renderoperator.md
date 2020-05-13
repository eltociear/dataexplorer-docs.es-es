---
title: 'operador de representación: Azure Explorador de datos'
description: En este artículo se describe el operador de representación en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 20e512d55568f39ea21d3ddcb383adaf0fa7dab3
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373033"
---
# <a name="render-operator"></a>Operador render

Indica al agente de usuario que represente los resultados de la consulta de una manera determinada.

```kusto
range x from 0.0 to 2*pi() step 0.01 | extend y=sin(x) | render linechart
```

> [!NOTE]
> * El operador Render debe ser el último operador de la consulta y solo se puede usar con consultas que producen un único resultado de flujo de datos tabular.
> * El operador Render no modifica los datos. Inserta una anotación ("visualización") en las propiedades extendidas del resultado. La anotación contiene la información proporcionada por el operador en la consulta.
> * El agente de usuario realiza la interpretación de la información de visualización. Los agentes diferentes (como Kusto. Explorer, Kusto. webexplorer) podrían admitir diferentes visualizaciones.

**Sintaxis**

*T* `|` `render` *visualización* [ `with` `(` *PropertyName* `=` *PropertyValue* [ `,` ...] `)` ]

Donde:

* La *visualización* indica el tipo de visualización que se va a usar. Los valores admitidos son:

::: zone pivot="azuredataexplorer"

|*Visualización*     |Descripción|
|--------------------|-|
| `anomalychart`     | Similar a gráfico, pero [resalta las anomalías](./samples.md#get-more-out-of-your-data-in-kusto-using-machine-learning) mediante [series_decompose_anomalies](./series-decompose-anomaliesfunction.md) función. |
| `areachart`        | Gráfico de área. La primera columna es el eje x y debe ser una columna numérica. Otras columnas numéricas son ejes y. |
| `barchart`         | La primera columna es el eje x y puede ser texto, fecha y hora o numérico. Otras columnas son numéricas, que se muestran como bandas horizontales.|
| `card`             | El primer registro de resultados se trata como un conjunto de valores escalares y se muestra como una tarjeta. |
| `columnchart`      | Como `barchart` con las tiras verticales en lugar de las franjas horizontales.|
| `ladderchart`      | Las dos últimas columnas son el eje x y otras columnas son eje y.|
| `linechart`        | gráfico de líneas. La primera columna es un eje x y debe ser una columna numérica. Otras columnas numéricas son ejes y. |
| `piechart`         | la primera columna es el eje de color y la segunda columna es numérica. |
| `pivotchart`       | Muestra un gráfico y una tabla dinámica. El usuario puede seleccionar interactivamente datos, columnas, filas y varios tipos de gráficos. |
| `scatterchart`     | Gráfico de puntos. La primera columna es un eje x y debe ser una columna numérica. Otras columnas numéricas son ejes y. |
| `stackedareachart` | Gráfico de áreas apilado. La primera columna es un eje x y debe ser una columna numérica. Otras columnas numéricas son ejes y. |
| `table`            | Predeterminado: los resultados se muestran como una tabla.|
| `timechart`        | gráfico de líneas. La primera columna es el eje x y debe ser la fecha y hora. Otras columnas (numéricas) son ejes y. Hay una columna de cadena cuyos valores se usan para "agrupar" las columnas numéricas y crear líneas diferentes en el gráfico (se omiten las columnas de cadena adicionales). |
| `timepivot`        | Navegación interactiva en la línea de tiempo de eventos (dinamización en el eje de tiempo)|

::: zone-end

::: zone pivot="azuremonitor"

|*Visualización*     |Descripción|
|--------------------|-|
| `areachart`        | Gráfico de área. La primera columna es el eje x y debe ser una columna numérica. Otras columnas numéricas son ejes y. |
| `barchart`         | La primera columna es el eje x y puede ser texto, fecha y hora o numérico. Otras columnas son numéricas, que se muestran como bandas horizontales.|
| `columnchart`      | Como `barchart` con las tiras verticales en lugar de las franjas horizontales.|
| `piechart`         | la primera columna es el eje de color y la segunda columna es numérica. |
| `scatterchart`     | Gráfico de puntos. La primera columna es el eje x y debe ser una columna numérica. Otras columnas numéricas son ejes y. |
| `table`            | Predeterminado: los resultados se muestran como una tabla.|
| `timechart`        | gráfico de líneas. La primera columna es el eje x y debe ser la fecha y hora. Otras columnas (numéricas) son ejes y. Hay una columna de cadena cuyos valores se usan para "agrupar" las columnas numéricas y crear líneas diferentes en el gráfico (se omiten las columnas de cadena adicionales).|

::: zone-end

* *NombreDePropiedad* / *PropertyValue* indica información adicional que se va a utilizar al representar.
  Todas las propiedades son opcionales. Las propiedades que se admiten son:

::: zone pivot="azuredataexplorer"

|*PropertyName*|*PropertyValue*                                                                   |
|--------------|----------------------------------------------------------------------------------|
|`accumulate`  |Si el valor de cada medida se agrega a todas sus predecesoras. ( `true` o `false` )|
|`kind`        |Más información sobre el tipo de visualización. Véase a continuación.                         |
|`legend`      |Indica si se va a mostrar una leyenda o no ( `visible` o `hidden` ).                       |
|`series`      |Lista delimitada por comas de columnas cuyos valores por registro combinados definen la serie a la que pertenece el registro.|
|`ymin`        |Valor mínimo que se va a mostrar en el eje Y.                                      |
|`ymax`        |Valor máximo que se va a mostrar en el eje Y.                                      |
|`title`       |Título de la visualización (de tipo `string` ).                                |
|`xaxis`       |Cómo escalar el eje x ( `linear` o `log` ).                                      |
|`xcolumn`     |La columna del resultado que se usa para el eje x.                                |
|`xtitle`      |Título del eje x (del tipo `string` ).                                       |
|`yaxis`       |Cómo escalar el eje y ( `linear` o `log` ).                                      |
|`ycolumns`    |Lista delimitada por comas de las columnas que se componen de los valores proporcionados por valor de la columna x.|
|`ysplit`      |Cómo dividir varias visualizaciones. Véase a continuación.                               |
|`ytitle`      |Título del eje y (de tipo `string` ).                                       |
|`anomalycolumns`|Propiedad relevante solo para `anomalychart` . Una lista delimitada por comas de las columnas que se considerarán como series de anomalías y se mostrarán como puntos en el gráfico|

::: zone-end

::: zone pivot="azuremonitor"

|*PropertyName*|*PropertyValue*                                                                   |
|--------------|----------------------------------------------------------------------------------|
|`kind`        |Más información sobre el tipo de visualización. Véase a continuación.                         |
|`series`      |Lista delimitada por comas de columnas cuyos valores por registro combinados definen la serie a la que pertenece el registro.|
|`title`       |Título de la visualización (de tipo `string` ).                                |
|`yaxis`       |Cómo escalar el eje y ( `linear` o `log` ).                                      |

::: zone-end

Algunas visualizaciones se pueden desarrollar aún más proporcionando la `kind` propiedad.
Estos son:

|*Visualización*|`kind`             |Descripción                        |
|---------------|-------------------|-----------------------------------|
|`areachart`    |`default`          |Cada "área" es independiente.     |
|               |`unstacked`        |Igual a `default`.                 |
|               |`stacked`          |Apilar "áreas" a la derecha.        |
|               |`stacked100`       |Apile "áreas" a la derecha y estire cada una de ellas al mismo ancho que los demás.|
|`barchart`     |`default`          |Cada "barra" se representa por sí misma.      |
|               |`unstacked`        |Igual a `default`.                 |
|               |`stacked`          |"Barras" de la pila.                      |
|               |`stacked100`       |Apilar "Bard" y ajustar cada uno de ellos al mismo ancho que los demás.|
|`columnchart`  |`default`          |Cada "columna" es independiente.   |
|               |`unstacked`        |Igual a `default`.                 |
|               |`stacked`          |Apile "columnas" una encima de la otra.|
|               |`stacked100`       |Apile "columnas" y estire cada una de ellas con el mismo alto que los demás.|
|`piechart`     |`map`              |Las columnas esperadas son [longitud, latitud] o punto GeoJSON, eje de color y numérico. Se admite en el escritorio de Kusto Explorer.|
|`scatterchart` |`map`              |Las columnas esperadas son [longitud, latitud] o punto de GeoJSON. La columna de la serie es opcional. Se admite en el escritorio de Kusto Explorer.|

::: zone pivot="azuredataexplorer"

Algunas visualizaciones admiten la división en varios valores del eje y:

|`ysplit`  |Descripción                                                       |
|----------|------------------------------------------------------------------|
|`none`    |Se muestra un solo eje y para todos los datos de la serie. (Es el valor predeterminado).       |
|`axes`    |Un solo gráfico se muestra con varios ejes y (uno por serie).|
|`panels`  |Un gráfico se representa para cada `ycolumn` valor (hasta un límite).|

::: zone-end

> [!NOTE]
> El modelo de datos del operador de representación examina los datos tabulares como si se tratase de tres tipos de columnas:
>
> * Columna del eje x (indicada por la `xcolumn` propiedad).
> * Columnas de la serie (cualquier número de columnas indicado por la `series` propiedad). Para cada registro, los valores combinados de estas columnas definen una sola serie y el gráfico tiene tantas series como valores combinados distintos.
> * Columnas del eje y (cualquier número de columnas indicado por la `ycolumns` propiedad).
  Para cada registro, la serie tiene tantas medidas ("puntos" en el gráfico) como columnas del eje y.

> [!TIP]
> 
> * Use `where` `summarize` y `top` para limitar el volumen que se muestra.
> * Ordene los datos para definir el orden del eje x.
> * Los agentes de usuario pueden "adivinar" el valor de las propiedades que no se especifican en la consulta. En concreto, tener columnas "sin intereses" en el esquema del resultado podría traducirse en una adivinación incorrecta. Pruebe a proyectar esas columnas cuando esto suceda. 

**Ejemplo**

<!-- csl: https://help.kusto.windows.net/Samples -->
```kusto
range x from -2 to 2 step 0.1
| extend sin = sin(x), cos = cos(x)
| extend x_sign = iif(x > 0, "x_pos", "x_neg")
| extend sum_sign = iif(sin + cos > 0, "sum_pos", "sum_neg")
| render linechart with  (ycolumns = sin, cos, series = x_sign, sum_sign)
```

::: zone pivot="azuredataexplorer"

[Ejemplos de representación en el tutorial](./tutorial.md#render-display-a-chart-or-table).

[Detección de anomalías](./samples.md#get-more-out-of-your-data-in-kusto-using-machine-learning)

::: zone-end
