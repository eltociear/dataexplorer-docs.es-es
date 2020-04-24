---
title: 'Operadores de cadena: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen los operadores String en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 91955ef5877e6c054917f70c655fc4b7cd3f277b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81516480"
---
# <a name="string-operators"></a>Operadores de cadena

En la tabla siguiente se resumen los operadores de las cadenas:

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
`has_any`       |Igual `has` que, pero funciona en cualquiera de los elementos                    |No            |`"North America" has_any("south", "north")`

## <a name="performance-tips"></a>Consejos de rendimiento

Para un mejor rendimiento, cuando haya dos operadores que realicen la misma tarea, utilice el que distingue mayúsculas de minúsculas.
Por ejemplo:

* en `=~`lugar de , use`==`
* en `in~`lugar de , use`in`
* en `contains`lugar de , use`contains_cs`

Para obtener resultados más rápidos, si está probando la presencia de un símbolo o palabra alfanumérica enlazada `has` `in`por caracteres no alfanuméricos (o el inicio o el final de un campo), utilice o . `has`realiza más `contains`rápido `startswith`que `endswith`, , o .

Por ejemplo, la primera de estas consultas se ejecuta más rápido:

```kusto
EventLog | where continent has "North" | count;
EventLog | where continent contains "nor" | count
```

## <a name="understanding-string-terms"></a>Comprender los términos de la cadena

De forma predeterminada, Kusto indexa todas `string`las columnas, incluidas las columnas de tipo .
De hecho, se crean varios índices para estas columnas, dependiendo de los datos reales. Para el usuario, estos índices no se exponen directamente (salvo su efecto `string` positivo en `has` el rendimiento de `has` `!has`las `hasprefix` `!hasprefix`consultas, por supuesto) con la excepción de los operadores que tienen como parte de su nombre: , , , , etc. Esos operadores son especiales en el sentido de que su semántica está dictada por la forma en que se codifica la columna; en lugar de hacer una coincidencia de subcadena "sencilla", estos operadores coinciden con **los términos**.

Para entender la coincidencia basada en términos, primero hay que entender lo que es un término. De forma predeterminada, Kusto divide cada `string` valor en secuencias máximas de caracteres alfanuméricos ASCII, y cada uno de ellos se convierte en un término. Por ejemplo, en `string`los siguientes `Kusto` `WilliamGates3rd`, los términos `ad67d136`son `c1db` `4f9f`, `88ef`, y las siguientes subcadenas: , , , , `d94f3b6b0b5a`:

```
Kusto:  ad67d136-c1db-4f9f-88ef-d94f3b6b0b5a;;WilliamGates3rd
```

De forma predeterminada, Kusto crea un índice de términos que consta de todos `has` `!has`los términos que son cuatro caracteres o más, y este índice es utilizado por , , etc. cuando se buscan términos que también son cuatro caracteres o más. (Si la consulta busca un término menor que cuatro `contains` caracteres o utiliza un operador, por ejemplo, Kusto volverá a examinar los valores de la columna si no puede determinar una coincidencia, que es mucho más lenta que buscar el término en el término index.)

