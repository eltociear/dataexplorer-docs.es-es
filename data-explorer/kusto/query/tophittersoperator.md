---
title: 'operador de los mejores bateadores: Explorador de datos de Azure Microsoft Docs'
description: En este artículo se describe el operador de los principales éxitos en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 7105bd4f9818deab1f0fa5dbe25dbb2d5b1b4afc
ms.sourcegitcommit: 436cd515ea0d83d46e3ac6328670ee78b64ccb05
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2020
ms.locfileid: "81663120"
---
# <a name="top-hitters-operator"></a>operador top-hitters

Devuelve una aproximación de los primeros *N* resultados (suponiendo la distribución sesgada de la entrada).

```kusto
T | top-hitters 25 of Page by Views 
```

**Sintaxis**

*T* `| top-hitters` *NumberOfRows* `of` *sort_key* `[` `by` *expresión*`]`

**Argumentos**

* *NumberOfRows*: el número de filas de *T* que se van a devolver. Puede especificar cualquier expresión numérica.
* *sort_key*: el nombre de la columna por la que se ordenan las filas.
* *expresión*: (opcional) Una expresión que se utilizará para la estimación de los mejores bateadores. 
    * *expression*: top-hitters devolverá *NumberOfRows* filas que tienen un máximo aproximado de sum(*expresión*). Expresión puede ser una columna o cualquier otra expresión que se evalúa como un número. 
    *  Si no se menciona la *expresión,* el algoritmo top-hitters contará las apariciones de la *clave de ordenación*.  

**Notas**

`top-hitters`es un algoritmo de aproximación y debe utilizarse cuando se ejecuta con datos de gran tamaño. La aproximación de los mejores bateadores se basa en el algoritmo [Count-Min-Sketch.](https://en.wikipedia.org/wiki/Count%E2%80%93min_sketch)  

**Ejemplo**

## <a name="getting-top-hitters-most-frequent-items"></a>Conseguir los mejores bateadores (artículos más frecuentes) 

El siguiente ejemplo muestra cómo encontrar los 5 idiomas principales con la mayoría de las páginas de Wikipedia (a las que se accedió después de abril de 2016). 

```kusto
PageViews
| where Timestamp > datetime(2016-04-01) and Timestamp < datetime(2016-05-01) 
| top-hitters 5 of Language 
```

|Idioma|approximate_count_Language|
|---|---|
|en|1539954127|
|zh|339827659|
|de|262197491|
|ru|227003107|
|fr|207943448|

## <a name="getting-top-hitters-based-on-column-value-"></a>Obtención de los mejores bateadores (basado en el valor de la columna) ***

El siguiente ejemplo muestra cómo encontrar las páginas en inglés más vistas de Wikipedia del año 2016. La consulta utiliza 'Vistas' (número entero) para calcular la popularidad de la página (número de vistas). 

```kusto
PageViews
| where Timestamp > datetime(2016-01-01)
| where Language == "en"
| where Page !has 'Special'
| top-hitters 10 of Page by Views
```

|Página|approximate_sum_Views|
|---|---|
|Main_Page|1325856754|
|Web_scraping|43979153|
|Java_(programming_language)|16489491|
|Estados_unidos_de_américa|13928841|
|Wikipedia|13584915|
|Donald_Trump|12376448|
|YouTube|11917252|
|The_Revenant_(2015_film)|10714263|
|Star_Wars:_The_Force_Awakens|9770653|
|Portal:Current_events|9578000|