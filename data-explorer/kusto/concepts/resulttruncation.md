---
title: 'El conjunto de resultados de la consulta Kusto supera el límite interno: Azure Explorador de datos'
description: En este artículo se describe el conjunto de resultados de la consulta se ha superado el valor interno... límite en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 9706cce08a99989441aa657c5d85465294be837e
ms.sourcegitcommit: 188f89553b9d0230a8e7152fa1fce56c09ebb6d6
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/08/2020
ms.locfileid: "84512425"
---
# <a name="query-result-set-has-exceeded-the-internal--limit"></a>El conjunto de resultados de la consulta ha superado el valor interno... ilimitado

Un *conjunto de resultados de la consulta ha superado el valor interno... Limit* es un tipo de [error de consulta parcial](partialqueryfailures.md) que se produce cuando el resultado de la consulta ha superado uno de dos límites:
* Un límite en el número de registros ( `record count limit` , establecido de forma predeterminada en 500.000)
* Un límite en la cantidad total de datos ( `data size limit` , establecido de forma predeterminada en 67.108.864 (64 MB)

Hay varios posibles cursos de acción:

* Cambie la consulta para consumir menos recursos. 
  Por ejemplo, puede:
  * Limitar el número de registros devueltos por la consulta mediante el [operador Take](../query/takeoperator.md) o agregar [cláusulas WHERE](../query/whereoperator.md) adicionales
  * Intente reducir el número de columnas devueltas por la consulta. Usar el [operador de proyecto](../query/projectoperator.md)o el [operador de ubicación de proyecto](../query/projectawayoperator.md))
  * Usar el [operador de Resumen](../query/summarizeoperator.md) para obtener datos agregados
* Aumente el límite de consulta pertinente temporalmente para esa consulta. Para obtener más información, vea **truncamiento del resultado** en límites de [consultas](querylimits.md)).

 > [!NOTE] 
 > No se recomienda aumentar el límite de consultas, ya que existen límites para proteger el clúster. Los límites se aseguran de que una sola consulta no interrumpa las consultas simultáneas que se ejecutan en el clúster.
  
