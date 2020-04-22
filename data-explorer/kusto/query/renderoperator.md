---
title: 'operador de representación: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describe el operador de representación en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/29/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 645860405a165f3304f655e42d2ced9b8777b8d0
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765475"
---
# <a name="render-operator"></a>Operador render

Indica al agente de usuario que represente los resultados de la consulta de una manera determinada.

```kusto
range x from 0.0 to 2*pi() step 0.01 | extend y=sin(x) | render linechart
```

> [!NOTE]
> * El operador de representación debe ser el último operador de la consulta y se usa solo con consultas que producen un único resultado de secuencia de datos tabular.
> * El operador de representación no modifica los datos. Inyecta una anotación ("Visualización") en las propiedades extendidas del resultado. La anotación contiene la información proporcionada por el operador en la consulta.
> * La interpretación de la información de visualización la realiza el agente de usuario. Diferentes agentes (como Kusto.Explorer,Kusto.WebExplorer) pueden admitir diferentes visualizaciones.

**Sintaxis**

*T* `|` `with` `(` *PropertyName* `=` *PropertyValue* `,` *Visualization* Visualización [ PropertyName PropertyValue [ ...] `render` `)`]

Donde:

* *La visualización* indica el tipo de visualización que se va a utilizar. Los valores admitidos son:

::: zone pivot="azuredataexplorer"

|*Visualización*     |Descripción|
|--------------------|-|
| `anomalychart`     | Similar al gráfico de tiempo, pero [resalta anomalías](./samples.md#get-more-out-of-your-data-in-kusto-using-machine-learning) utilizando [series_decompose_anomalies](./series-decompose-anomaliesfunction.md) función. |
| `areachart`        | Gráfico de área. La primera columna es el eje X y debe ser una columna numérica. Otras columnas numéricas son ejes Y. |
| `barchart`         | La primera columna es el eje X y puede ser texto, datetime o numérico. Otras columnas son numéricas, que se muestran como tiras horizontales.|
| `card`             | El primer registro de resultado se trata como un conjunto de valores escalares y se muestra como una tarjeta. |
| `columnchart`      | Como `barchart` con tiras verticales en lugar de tiras horizontales.|
| `ladderchart`      | Las dos últimas columnas son el eje X, otras columnas son del eje Y.|
| `linechart`        | gráfico de líneas. La primera columna es del eje X y debe ser una columna numérica. Otras columnas numéricas son ejes Y. |
| `piechart`         | la primera columna es el eje de color y la segunda columna es numérica. |
| `pivotchart`       | Muestra una tabla dinámica y un gráfico. El usuario puede seleccionar de forma interactiva datos, columnas, filas y varios tipos de gráficos. |
| `scatterchart`     | Gráfico de puntos. La primera columna es del eje X y debe ser una columna numérica. Otras columnas numéricas son ejes Y. |
| `stackedareachart` | Gráfico de área apilada. La primera columna es del eje X y debe ser una columna numérica. Otras columnas numéricas son ejes Y. |
| `table`            | Predeterminado: los resultados se muestran como una tabla.|
| `timechart`        | gráfico de líneas. La primera columna es el eje x y debe ser la fecha y hora. Otras columnas (numéricas) son ejes Y. Hay una columna de cadena cuyos valores se utilizan para "agrupar" las columnas numéricas y crear diferentes líneas en el gráfico (se omiten más columnas de cadena). |
| `timepivot`        | Navegación interactiva a través de la línea de tiempo de eventos (pivote en el eje de tiempo)|

::: zone-end

::: zone pivot="azuremonitor"

|*Visualización*     |Descripción|
|--------------------|-|
| `areachart`        | Gráfico de área. La primera columna es el eje X y debe ser una columna numérica. Otras columnas numéricas son ejes Y. |
| `barchart`         | La primera columna es el eje X y puede ser texto, datetime o numérico. Otras columnas son numéricas, que se muestran como tiras horizontales.|
| `columnchart`      | Como `barchart` con tiras verticales en lugar de tiras horizontales.|
| `piechart`         | la primera columna es el eje de color y la segunda columna es numérica. |
| `scatterchart`     | Gráfico de puntos. La primera columna es el eje X y debe ser una columna numérica. Otras columnas numéricas son ejes Y. |
| `table`            | Predeterminado: los resultados se muestran como una tabla.|
| `timechart`        | gráfico de líneas. La primera columna es el eje x y debe ser la fecha y hora. Otras columnas (numéricas) son ejes Y. Hay una columna de cadena cuyos valores se utilizan para "agrupar" las columnas numéricas y crear diferentes líneas en el gráfico (se omiten más columnas de cadena).|

::: zone-end

* *PropertyName*/*PropertyValue* indica información adicional que se debe usar al representar.
  Todas las propiedades son opcionales. Las propiedades que se admiten son:

::: zone pivot="azuredataexplorer"

|*Propertyname*|*Valor_propiedad*                                                                   |
|--------------|----------------------------------------------------------------------------------|
|`accumulate`  |Si el valor de cada medida se agrega a todos sus predecesores. (`true` `false`o )|
|`kind`        |Elaboración adicional del tipo de visualización. Véase a continuación.                         |
|`legend`      |Si desea mostrar una`visible` leyenda `hidden`o no ( o ).                       |
|`series`      |Lista delimitada por comas de columnas cuyos valores combinados por registro definen la serie a la que pertenece el registro.|
|`ymin`        |El valor mínimo que se mostrará en el eje Y.                                      |
|`ymax`        |El valor máximo que se mostrará en el eje Y.                                      |
|`title`       |El título de la `string`visualización (de tipo ).                                |
|`xaxis`       |Cómo escalar el eje`linear` x `log`( o ).                                      |
|`xcolumn`     |Qué columna del resultado se utiliza para el eje X.                                |
|`xtitle`      |El título del eje X `string`(de tipo ).                                       |
|`yaxis`       |Cómo escalar el eje`linear` Y `log`( o ).                                      |
|`ycolumns`    |Lista delimitada por comas de columnas que constan de los valores proporcionados por valor de la columna x.|
|`ysplit`      |Cómo dividir varias la visualización. Véase a continuación.                               |
|`ytitle`      |El título del eje Y `string`(de tipo ).                                       |
|`anomalycolumns`|Propiedad relevante `anomalychart`solo para . Lista delimitada por comas de columnas que se considerarán como series de anomalías y se mostrarán como puntos en el gráfico|

::: zone-end

::: zone pivot="azuremonitor"

|*Propertyname*|*Valor_propiedad*                                                                   |
|--------------|----------------------------------------------------------------------------------|
|`kind`        |Elaboración adicional del tipo de visualización. Véase a continuación.                         |
|`series`      |Lista delimitada por comas de columnas cuyos valores combinados por registro definen la serie a la que pertenece el registro.|
|`title`       |El título de la `string`visualización (de tipo ).                                |
|`yaxis`       |Cómo escalar el eje`linear` Y `log`( o ).                                      |

::: zone-end

Algunas visualizaciones se pueden elaborar `kind` más detalladamente proporcionando la propiedad.
Estos son:

|*Visualización*|`kind`             |Descripción                        |
|---------------|-------------------|-----------------------------------|
|`areachart`    |`default`          |Cada "área" se encuentra por sí sola.     |
|               |`unstacked`        |Igual a `default`.                 |
|               |`stacked`          |Apilar "áreas" a la derecha.        |
|               |`stacked100`       |Apilar "áreas" a la derecha y estirar cada una a la misma anchura que las otras.|
|`barchart`     |`default`          |Cada "bar" se encuentra por sí solo.      |
|               |`unstacked`        |Igual a `default`.                 |
|               |`stacked`          |Apilar "barras".                      |
|               |`stacked100`       |Apilar "bard" y estirar cada uno a la misma anchura que los demás.|
|`columnchart`  |`default`          |Cada "columna" se encuentra por sí sola.   |
|               |`unstacked`        |Igual a `default`.                 |
|               |`stacked`          |Apilar "columnas" una encima de la otra.|
|               |`stacked100`       |Apilar "columnas" y estirar cada una a la misma altura que las otras.|
|`piechart`     |`map`              |Las columnas esperadas son [Longitud, Latitud] o Punto GeoJSON, eje de color y numérico. Compatible con el escritorio de Kusto Explorer.|
|`scatterchart` |`map`              |Las columnas esperadas son [Longitud, Latitud] o punto GeoJSON. La columna de la serie es opcional. Compatible con el escritorio de Kusto Explorer.|

::: zone pivot="azuredataexplorer"

Algunas visualizaciones admiten la división en varios valores del eje Y:

|`ysplit`  |Descripción                                                       |
|----------|------------------------------------------------------------------|
|`none`    |Se muestra un único eje Y para todos los datos de la serie. (Es el valor predeterminado).       |
|`axes`    |Se muestra un único gráfico con varios ejes Y (uno por serie).|
|`panels`  |Se representa un gráfico `ycolumn` para cada valor (hasta algún límite).|

::: zone-end

> [!NOTE]
> El modelo de datos del operador de representación examina los datos tabulares como si tiene tres tipos de columnas:
>
> * La columna del eje x `xcolumn` (indicada por la propiedad).
> * Las columnas de serie (cualquier número `series` de columnas indicadas por la propiedad.) Para cada registro, los valores combinados de estas columnas definen una sola serie y el gráfico tiene tantas series como valores combinados distintos.
> * Las columnas del eje y (cualquier `ycolumns` número de columnas indicadas por la propiedad).
  Para cada registro, la serie tiene tantas medidas ("puntos" en el gráfico) como columnas del eje Y.

> [!TIP]
> 
> * Utilice `where` `summarize` , `top` y para limitar el volumen que muestra.
> * Ordene los datos para definir el orden del eje X.
> * Los agentes de usuario son libres de "adivinar" el valor de las propiedades que no se especifican en la consulta. En particular, tener columnas "poco interesantes" en el esquema del resultado podría traducirse en ellas adivinando mal. Intente proyectar de sano tales columnas cuando eso suceda. 

**Ejemplo**

```kusto
range x from -2 to 2 step 0.1
| extend sin = sin(x), cos = cos(x)
| extend x_sign = iif(x > 0, "x_pos", "x_neg")
| extend sum_sign = iif(sin + cos > 0, "sum_pos", "sum_neg")
| render linechart with  (ycolumns = sin, cos, series = x_sign, sum_sign)
```

::: zone pivot="azuredataexplorer"

[Representar ejemplos en el tutorial](./tutorial.md#render-display-a-chart-or-table).

[Detección de anomalías](./samples.md#get-more-out-of-your-data-in-kusto-using-machine-learning)

::: zone-end
