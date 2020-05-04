---
title: 'Instrucciones de consulta: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las instrucciones de consulta de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 12/10/2019
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 6e383ce9fdcf373452c0b7d710302669e7987395
ms.sourcegitcommit: d885c0204212dd83ec73f45fad6184f580af6b7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82737851"
---
# <a name="query-statements"></a>Instrucciones de consulta

::: zone pivot="azuredataexplorer"

Una consulta consta de una o varias **instrucciones de consulta**, delimitadas por un punto`;`y coma ().
Al menos una de estas instrucciones de consulta debe ser una [instrucción de expresión tabular](./tabularexpressionstatements.md).
La instrucción de expresión tabular genera uno o varios resultados tabulares.
Cuando la consulta tiene más de una instrucción de expresión tabular, la consulta tiene un [lote](./batches.md) de instrucciones de expresión tabular y los resultados tabulares generados por estas instrucciones son devueltos por la consulta.

Dos tipos de instrucciones de consulta:

* Instrucciones que utilizan principalmente los usuarios ([instrucciones de consulta de usuario](#user-query-statements)),
* Instrucciones diseñadas para admitir escenarios en los que las aplicaciones de nivel intermedio toman consultas de usuario y envían una versión modificada de ellas a Kusto ([instrucciones de consulta de aplicación](#application-query-statements)).

Algunas instrucciones de consulta son útiles en ambos escenarios.

> [!NOTE]
> El "efecto" de una instrucción de consulta se inicia en el punto en el que aparece la instrucción en la consulta y termina al final de la consulta. Una vez completada la consulta, se liberan todos sus recursos y no tiene ningún impacto en las consultas futuras (a excepción de los efectos secundarios, como hacer que la consulta se grabe en un registro de todas las consultas ejecutadas o tener sus resultados almacenados en caché).

## <a name="user-query-statements"></a>Instrucciones de consulta de usuario

A continuación se muestra una lista de instrucciones de consulta de usuario:

* Una [instrucción Let](./letstatement.md) define un enlace entre un nombre y una expresión.
  Las instrucciones Let se pueden usar para dividir una consulta larga en pequeñas partes con nombre que son más fáciles de entender.

* Una [instrucción set](./setstatement.md) establece una opción de consulta que afecta al modo en que se procesa la consulta y se devuelven sus resultados.

* Una [instrucción de expresión tabular](./tabularexpressionstatements.md), la instrucción de consulta más importante, devuelve los datos "interesantes" como resultados.

## <a name="application-query-statements"></a>Instrucciones de consulta de aplicaciones

A continuación se muestra una lista de instrucciones de consulta de aplicación:

* Una [instrucción de alias](./aliasstatement.md) define un alias para otra base de datos (en el mismo clúster o en un clúster remoto).

* Una [instrucción Pattern](./patternstatement.md), que pueden usar las aplicaciones que se basan en Kusto y que exponen el lenguaje de consulta a sus usuarios para insertarse en el proceso de resolución de nombres de consulta.

* Una [instrucción de parámetros de consulta](./queryparametersstatement.md), que usan las aplicaciones que se basan en Kusto para protegerse frente a ataques de inyección (similar a cómo los parámetros de comando protegen SQL frente a ataques de inyección de SQL).

* [Instrucción Restrict](./restrictstatement.md), que usan las aplicaciones que se basan en Kusto para restringir las consultas a un subconjunto específico de datos de Kusto (incluida la restricción del acceso a columnas y registros específicos).

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end
