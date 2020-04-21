---
title: 'Errores parciales de la consulta: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describen los errores de consulta parciales en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: d65256c0ac50a80e3e3b095e8d2348dbae5e585b
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523127"
---
# <a name="partial-query-failures"></a>Errores de consulta parcial

Un error parcial de *la consulta* es un error al ejecutar la consulta que se detecta solo después de que la consulta haya iniciado la fase de ejecución real. En ese momento, Kusto ya `200 OK` ha devuelto la línea de estado HTTP al cliente y no puede "recuperarla", por lo que tiene que indicar el error en la secuencia de resultados que lleva los resultados de la consulta de vuelta al cliente. (De hecho, es posible que ya haya devuelto algunos datos de resultados al autor de la llamada.)

Hay varios tipos de errores de consulta parciales:
* [Consultas de ejecución:](runawayqueries.md)consultas que toman demasiados recursos.
* [Truncamiento](resulttruncation.md)de resultados : Consultas cuyo conjunto de resultados se ha truncado ya que excedió algún límite.
* [Desbordamientos](overflow.md): Consultas que desencadenan un error de desbordamiento.

Los errores de consulta parcial es de vuelta al cliente de una de estas dos maneras:

* Como parte de los datos de resultado (un tipo especial de registro indica que no se trata de los datos en sí). Esta es la forma predeterminada.
* Como parte de la tabla "QueryStatus" en la secuencia de resultados. Esto se hace `deferpartialqueryfailures` mediante la opción `properties` en`Kusto.Data.Common.ClientRequestProperties.OptionDeferPartialQueryFailures`la ranura de la solicitud ( ).
  Los clientes que lo hacen asumen la responsabilidad de `QueryStatus` consumir toda la secuencia de resultados `Severity` del `2` servicio, localizar el resultado y asegurarse de que ningún registro en este resultado tiene uno de o más pequeño. 