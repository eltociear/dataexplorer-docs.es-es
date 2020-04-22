---
title: 'Instrucciones de consulta: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen las instrucciones Query en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/10/2019
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 20eeb1aa755fcf4e3068cba061a2738a375e1847
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765566"
---
# <a name="query-statements"></a>Instrucciones de consulta

::: zone pivot="azuredataexplorer"

Una consulta consta de una o varias instrucciones de`;` **consulta, delimitadas**por un punto y coma ( ).
Al menos una de estas instrucciones de consulta debe ser una [instrucción de expresión tabular.](./tabularexpressionstatements.md)
La instrucción de expresión tabular genera uno o varios resultados tabulares.
Cuando la consulta tiene más de una instrucción de expresión tabular, la consulta tiene un [lote](./batches.md) de instrucciones de expresión tabular y la consulta devuelve todos los resultados tabulares generados por estas instrucciones.

Dos tipos de instrucciones de consulta:

* Instrucciones que utilizan principalmente los[usuarios](#user-query-statements)( instrucciones de consulta de usuario ),
* Instrucciones que se han diseñado para admitir escenarios en los que las aplicaciones de nivel intermedio toman consultas de usuario y envían una versión modificada de ellas a Kusto ( instrucciones de consulta de[aplicación).](#application-query-statements)

Algunas instrucciones de consulta son útiles en ambos escenarios.

> [!NOTE]
> El "efecto" de una instrucción de consulta comienza en el punto donde la instrucción aparece en la consulta y termina al final de la consulta. Una vez completada la consulta, se liberan todos sus recursos y no tiene ningún impacto en las consultas futuras (excepto los efectos secundarios, como tener la consulta registrada en un registro de todas las consultas en ejecución o tener sus resultados almacenados en caché.)

## <a name="user-query-statements"></a>Instrucciones de consulta de usuario

A continuación se muestra una lista de instrucciones de consulta de usuario:

* Una [instrucción let](./letstatement.md) define un enlace entre un nombre y una expresión.
  Las instrucciones Let se pueden usar para dividir una consulta larga en pequeñas partes con nombre que son más fáciles de entender.

* Una [instrucción set](./setstatement.md) establece una opción de consulta que afecta a cómo se procesa la consulta y se devuelven sus resultados.

* Una [instrucción de expresión tabular](./tabularexpressionstatements.md), la instrucción de consulta más importante, devuelve los datos "interesantes" como resultados.

## <a name="application-query-statements"></a>Declaraciones de consulta de aplicación

A continuación se muestra una lista de instrucciones de consulta de aplicación:

* Una [instrucción alias](./aliasstatement.md) define un alias para otra base de datos (en el mismo clúster o en un clúster remoto).

* Una instrucción de [patrón,](./patternstatement.md)que pueden usar las aplicaciones creadas sobre Kusto y exponen el lenguaje de consulta a sus usuarios para insertarse en el proceso de resolución de nombres de consulta.

* Una instrucción de parámetros de [consulta,](./queryparametersstatement.md)que utilizan las aplicaciones creadas sobre Kusto para protegerse contra ataques de inyección (similar a cómo los parámetros de comando protegen SQL contra ataques de inyección SQL.)

* Una [instrucción restrict](./restrictstatement.md), que usan las aplicaciones creadas sobre Kusto para restringir las consultas a un subconjunto específico de datos en Kusto (incluida la restricción del acceso a columnas y registros específicos.)

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end
