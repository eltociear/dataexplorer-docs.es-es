---
title: 'Operadores de cadena: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen los operadores de cadena en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: e16d90c8307392536b5971758ce7df15a9ea1b46
ms.sourcegitcommit: ef009294b386cba909aa56d7bd2275a3e971322f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/08/2020
ms.locfileid: "82977140"
---
# <a name="string-operators"></a>Operadores de cadena

En la tabla siguiente se resumen los operadores de cadenas:

Operator        |Descripción                                                       |Distingue mayúsculas de minúsculas|Ejemplo (produce `true`)
----------------|------------------------------------------------------------------|--------------|-----------------------
`==`            |Equals                                                            |Sí           |`"aBc" == "aBc"`
`!=`            |Not Equals                                                        |Sí           |`"abc" != "ABC"`
`=~`            |Equals                                                            |No            |`"abc" =~ "ABC"`
`!~`            |Not Equals                                                        |No            |`"aBc" !~ "xyz"`
`has`           |El lado derecho (RHS) es un término completo en el lado izquierdo (LHS)     |No            |`"North America" has "america"`
`!has`          |RHS no es un término completo en LHS                                     |No            |`"North America" !has "amer"` 
`has_cs`        |El lado derecho (RHS) es un término completo en el lado izquierdo (LHS)     |Sí           |`"North America" has_cs "America"`
`!has_cs`       |RHS no es un término completo en LHS                                     |Sí           |`"North America" !has_cs "amer"` 
`hasprefix`     |RHS es un prefijo de término en LHS                                       |No            |`"North America" hasprefix "ame"`
`!hasprefix`    |RHS no es un prefijo de término en LHS                                   |No            |`"North America" !hasprefix "mer"` 
`hasprefix_cs`  |RHS es un prefijo de término en LHS                                       |Sí           |`"North America" hasprefix_cs "Ame"`
`!hasprefix_cs` |RHS no es un prefijo de término en LHS                                   |Sí           |`"North America" !hasprefix_cs "CA"` 
`hassuffix`     |RHS es un sufijo de término en LHS                                       |No            |`"North America" hassuffix "ica"`
`!hassuffix`    |RHS no es un sufijo de término en LHS                                   |No            |`"North America" !hassuffix "americ"`
`hassuffix_cs`  |RHS es un sufijo de término en LHS                                       |Sí           |`"North America" hassuffix_cs "ica"`
`!hassuffix_cs` |RHS no es un sufijo de término en LHS                                   |Sí           |`"North America" !hassuffix_cs "icA"`
`contains`      |RHS ocurre como una subsecuencia de LHS                                |No            |`"FabriKam" contains "BRik"`
`!contains`     |RHS no ocurre en LHS                                         |No            |`"Fabrikam" !contains "xyz"`
`contains_cs`   |RHS ocurre como una subsecuencia de LHS                                |Sí           |`"FabriKam" contains_cs "Kam"`
`!contains_cs`  |RHS no ocurre en LHS                                         |Sí           |`"Fabrikam" !contains_cs "Kam"`
`startswith`    |RHS es una subsecuencia inicial de LHS                              |No            |`"Fabrikam" startswith "fab"`
`!startswith`   |RHS no es una subsecuencia inicial de LHS                          |No            |`"Fabrikam" !startswith "kam"`
`startswith_cs` |RHS es una subsecuencia inicial de LHS                              |Sí           |`"Fabrikam" startswith_cs "Fab"`
`!startswith_cs`|RHS no es una subsecuencia inicial de LHS                          |Sí           |`"Fabrikam" !startswith_cs "fab"`
`endswith`      |RHS es una subsecuencia de cierre de LHS                               |No            |`"Fabrikam" endswith "Kam"`
`!endswith`     |RHS no es una subsecuencia de cierre de LHS                           |No            |`"Fabrikam" !endswith "brik"`
`endswith_cs`   |RHS es una subsecuencia de cierre de LHS                               |Sí           |`"Fabrikam" endswith "Kam"`
`!endswith_cs`  |RHS no es una subsecuencia de cierre de LHS                           |Sí           |`"Fabrikam" !endswith "brik"`
`matches regex` |LHS contiene una coincidencia para RHS                                      |Sí           |`"Fabrikam" matches regex "b.*k"`
`in`            |Igual a uno de los elementos                                     |Sí           |`"abc" in ("123", "345", "abc")`
`!in`           |No es igual a uno de los elementos                                 |Sí           |`"bca" !in ("123", "345", "abc")`
`in~`           |Igual a uno de los elementos                                     |No            |`"abc" in~ ("123", "345", "ABC")`
`!in~`          |No es igual a uno de los elementos                                 |No            |`"bca" !in~ ("123", "345", "ABC")`
`has_any`       |Igual que `has` pero funciona en cualquiera de los elementos                    |No            |`"North America" has_any("south", "north")`

## <a name="performance-tips"></a>Consejos de rendimiento

Para obtener un mejor rendimiento, cuando hay dos operadores que realizan la misma tarea, use la distinción entre mayúsculas y minúsculas.
Por ejemplo:

* en lugar de `=~`, use`==`
* en lugar de `in~`, use`in`
* en lugar de `contains`, use`contains_cs`

Para obtener resultados más rápidos, si está probando la presencia de un símbolo o una palabra alfanumérica enlazada por caracteres no alfanuméricos (o por el inicio o el final de un campo) `has` , `in`use o. `has`se realiza más `contains`rápido `startswith`que, `endswith`o.

Por ejemplo, la primera de estas consultas se ejecuta más rápido:

```kusto
EventLog | where continent has "North" | count;
EventLog | where continent contains "nor" | count
```

## <a name="understanding-string-terms"></a>Descripción de los términos de cadena

De forma predeterminada, Kusto indiza todas las columnas, incluidas las columnas `string`de tipo.
De hecho, se generan varios índices para estas columnas, en función de los datos reales. Para el usuario, estos índices no se exponen directamente (aparte de su efecto positivo en el rendimiento de las consultas del curso) con `string` la excepción de `has` los operadores que tienen como parte `has`de `!has`su `hasprefix`nombre `!hasprefix`:,,,, etc. Estos operadores son especiales, ya que su semántica viene determinada por la forma en que se codifica la columna; en lugar de realizar una coincidencia de subcadena "sin formato", estos operadores coinciden con los **términos**.

Para comprender la coincidencia basada en términos, primero debe comprender qué es un término. De forma predeterminada, Kusto divide `string` cada valor en secuencias máximas de caracteres alfanuméricos ASCII, y cada uno de ellos se convierte en un término. Por ejemplo, a continuación `string`, los términos son `Kusto`, `WilliamGates3rd`y las subcadenas siguientes: `ad67d136`, `c1db`, `4f9f`, `88ef`,: `d94f3b6b0b5a`

```
Kusto:  ad67d136-c1db-4f9f-88ef-d94f3b6b0b5a;;WilliamGates3rd
```

De forma predeterminada, Kusto crea un índice de términos que consta de todos los términos que tienen **cuatro caracteres o más**, y este índice `has`lo `!has`usa,, etc. al buscar términos que también son cuatro caracteres o más. (Si la consulta busca un término de menos de cuatro caracteres, o usa un `contains` operador por ejemplo, Kusto volverá a examinar los valores de la columna si no puede determinar una coincidencia, lo que es mucho más lento que buscar el término en el índice del término).

