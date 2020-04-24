---
title: 'operador de búsqueda: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe el operador de búsqueda en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: de63aa3fde421996809334b8a746ea4dee8cb026
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509204"
---
# <a name="search-operator"></a>Operador search

El operador de búsqueda proporciona una experiencia de búsqueda de varias tablas o varias columnas.

## <a name="syntax"></a>Sintaxis

* [*TabularSource* `|`] `search` [`kind=`*CaseSensitivity*`in` `(`] [ *TableSources*`)`] *SearchPredicate*

## <a name="arguments"></a>Argumentos

* *TabularSource*: una expresión tabular opcional que actúa como un origen de datos que se va a buscar, como un nombre de tabla, un operador de [unión,](unionoperator.md)los resultados de una consulta tabular, etc. No puede aparecer junto con la frase opcional que incluye *TableSources*.

* *CaseSensitivity*: Indicador opcional que controla el comportamiento de todos los `string` operadores escalares con respecto a la sensibilidad a mayúsculas y minúsculas. Los valores válidos `default` son `case_insensitive` los dos sinónimos y `has`(que es el valor `case_sensitive` predeterminado para operadores como , a saber, no distingue mayúsculas de minúsculas) y (que fuerza a todos estos operadores en el modo de coincidencia que distingue mayúsculas de minúsculas).

* *TableSources*: Una lista opcional separada por comas de nombres de tabla "comodín" para participar en la búsqueda.
  La lista tiene la misma sintaxis que la lista del [operador union](unionoperator.md).
  No puede aparecer junto con *tabularSource*opcional .

* *SearchPredicate*: Un predicado obligatorio que define qué buscar (en otras palabras, una expresión booleana `true`que se evalúa para cada registro de la entrada y que, si devuelve , se genera el registro.) La sintaxis de *SearchPredicate* amplía y modifica la sintaxis normal de Kusto para expresiones booleanas:

  **Extensiones**de coincidencia de cadena: los literales de cadena que aparecen `has`como `hasprefix` `hassuffix`términos en *SearchPredicate* indican una coincidencia de términos entre todas las columnas y el literal mediante , , , y las versiones invertidas (`!`) o que distinguen mayúsculas de minúsculas (`sc`) de estos operadores. La decisión `has`de `hasprefix`si `hassuffix` se aplica , , o depende de si`*`el literal comienza o termina (o ambos) mediante un asterisco ( ). No se permiten asteriscos dentro del literal.

    |Literal   |Operator   |
    |----------|-----------|
    |`billg`   |`has`      |
    |`*billg`  |`hassuffix`|
    |`billg*`  |`hasprefix`|
    |`*billg*` |`contains` |
    |`bi*lg`   |`matches regex`|

  **Restricción**de columnas: de forma predeterminada, las extensiones de coincidencia de cadenas intentan hacer coincidir con todas las columnas del conjunto de datos. Es posible restringir esta coincidencia a una columna determinada mediante la sintaxis siguiente: *ColumnName*`:`*StringLiteral*.

  **Igualdad**de cadenas: las coincidencias exactas de una columna con un valor de cadena (en lugar de una coincidencia de términos) se pueden realizar mediante la sintaxis *ColumnName*`==`*StringLiteral*.

  **Otras expresiones booleanas:** la sintaxis admite todas las expresiones booleanas de Kusto normales.
    Por `"error" and x==123` ejemplo, significa: busque registros `error` que tengan el término en `123` cualquiera `x` de sus columnas y tengan el valor en la columna."

  **Coincidencia de Regex:** la coincidencia de expresiones regulares se indica mediante la sintaxis *Column* `matches regex` *StringLiteral,* donde *StringLiteral* es el patrón regex.

Tenga en cuenta que si se omiten *TabularSource* y *TableSource,* la búsqueda se lleva a cabo todas las tablas y vistas sin restricciones de la base de datos en el ámbito.

## <a name="summary-of-string-matching-extensions"></a>Resumen de extensiones de coincidencia de cadenas

  |# |Sintaxis                                 |Significado (equivalente) `where`           |Comentarios|
  |--|---------------------------------------|---------------------------------------|--------|
  | 1|`search "err"`                         |`where * has "err"`                    ||
  | 2|`search in (T1,T2,A*) and "err"`       |<code>union T1,T2,A* &#124; where * has "err"<code>   ||
  | 3|`search col:"err"`                     |`where col has "err"`                  ||
  | 4|`search col=="err"`                    |`where col=="err"`                     ||
  | 5|`search "err*"`                        |`where * hasprefix "err"`              ||
  | 6|`search "*err"`                        |`where * hassuffix "err"`              ||
  | 7|`search "*err*"`                       |`where * contains "err"`               ||
  | 8|`search "Lab*PC"`                      |`where * matches regex @"\bLab\w*PC\b"`||
  | 9|`search *`                             |`where 0==0`                           ||
  |10|`search col matches regex "..."`       |`where col matches regex "..."`        ||
  |11|`search kind=case_sensitive`           |                                       |Todas las comparaciones de cadenas distinguen mayúsculas de minúsculas|
  |12|`search "abc" and ("def" or "hij")`    |`where * has "abc" and (* has "def" or * has hij")`||
  |13|`search "err" or (A>a and A<b)`        |`where * has "err" or (A>a and A<b)`   ||

## <a name="remarks"></a>Observaciones

**A diferencia** del `search` operador [find](findoperator.md), el operador no admite lo siguiente:

1. `withsource=`: la salida siempre incluirá `$table` una `string` columna llamada de tipo cuyo valor es el nombre de tabla del que se recuperó cada registro (o algún nombre generado por el sistema si el origen no es una tabla, sino una expresión compuesta).
2. `project=`, `project-smart`: el esquema `project-smart` de salida es equivalente al esquema de salida.

## <a name="examples"></a>Ejemplos

```kusto
// 1. Simple term search over all unrestricted tables and views of the database in scope
search "billg"

// 2. Like (1), but looking only for records that match both terms
search "billg" and ("steveb" or "satyan")

// 3. Like (1), but looking only in the TraceEvent table
search in (TraceEvent) and "billg"

// 4. Like (2), but performing a case-sensitive match of all terms
search "BillB" and ("SteveB" or "SatyaN")

// 5. Like (1), but restricting the match to some columns
search CEO:"billg" or CSA:"billg"

// 6. Like (1), but only for some specific time limit
search "billg" and Timestamp >= datetime(1981-01-01)

// 7. Searches over all the higher-ups
search in (C*, TF) "billg" or "davec" or "steveb"

// 8. A different way to say (7). Prefer to use (7) when possible
union C*, TF | search "billg" or "davec" or "steveb"
```

## <a name="performance-tips"></a>Sugerencias para mejorar el rendimiento

  |# |Sugerencia                                                                                  |Prefer                                        |Over                                                                    |
  |--|-------------------------------------------------------------------------------------|----------------------------------------------|------------------------------------------------------------------------|
  | 1| Prefiere utilizar un `search` solo operador `search` en varios operadores consecutivos|`search "billg" and ("steveb" or "satyan")`   |<code>search "billg" &#124; search "steveb" or "satyan"<code>           ||
  | 2| Prefiere filtrar dentro `search` del operador                                       |`search "billg" and "steveb"`                 |<code>search * &#124; where * has "billg" and * has "steveb"<code>      ||
