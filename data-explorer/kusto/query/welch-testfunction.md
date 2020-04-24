---
title: welch_test() - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe welch_test() en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a9f4b3d86bf3d679fd4fdd320b394956c71d3e97
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81504495"
---
# <a name="welch_test"></a>welch_test()

Calcula el p_value de la [función Welch-test](https://en.wikipedia.org/wiki/Welch%27s_t-test)

```kusto
// s1, s2 values are from https://en.wikipedia.org/wiki/Welch%27s_t-test
print
    s1 = dynamic([27.5, 21.0, 19.0, 23.6, 17.0, 17.9, 16.9, 20.1, 21.9, 22.6, 23.1, 19.6, 19.0, 21.7, 21.4]),
    s2 = dynamic([27.1, 22.0, 20.8, 23.4, 23.4, 23.5, 25.8, 22.0, 24.8, 20.2, 21.9, 22.1, 22.9, 20.5, 24.4])
| mv-expand s1 to typeof(double), s2 to typeof(double)
| summarize m1=avg(s1), v1=variance(s1), c1=count(), m2=avg(s2), v2=variance(s2), c2=count()
| extend pValue=welch_test(m1,v1,c1,m2,v2,c2)

// pValue = 0.021
```

**Sintaxis**

`welch_test(`*media1*`, `*varianza1*`, `*recuento1*`, `*media2*`, `*varianza2*`, `*conteo 2*`)`

**Argumentos**

* *mean1*: Expresión que representa el valor medio (promedio) de la 1a serie
* *variance1*: Expresión que representa el valor de varianza de la 1a serie
* *count1*: Expresión que representa el recuento de valores de la 1a serie
* *mean2*: Expresión que representa el valor medio (promedio) de la 2a serie
* *variance2*: Expresión que representa el valor de varianza de la 2a serie
* *count2*: Expresión que representa el recuento de valores de la 2a serie

**Devuelve**

De [Wikipedia](https://en.wikipedia.org/wiki/Welch%27s_t-test):

En las estadísticas, la prueba t de Welch, o la prueba t de varianzas desiguales, es una prueba de ubicación de dos muestras que se utiliza para probar la hipótesis de que dos poblaciones tienen medios iguales. La prueba t de Welch es una adaptación de la prueba t de Student, es decir, se ha derivado con la ayuda de la prueba t de Student y es más confiable cuando las dos muestras tienen varianzas desiguales y tamaños de muestra desiguales. Estas pruebas se conocen a menudo como "muestras no emparejadas" o "muestras independientes", ya que normalmente se aplican cuando las unidades estadísticas subyacentes a las dos muestras que se comparan no se superponen. Dado que la prueba t de Welch ha sido menos popular que la prueba t de Student y puede ser menos familiar para los lectores, un nombre más informativo es "Prueba t de varianzas desiguales de Welch" o "varianzas desiguales t-test" para la brevedad.