---
title: 'Consultas de sin ejecutar: Explorador de azure Data Explorer Microsoft Docs'
description: En este artículo se describen las consultas de ejecución en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 076afea64cf3c4405f37e9e4014584b90d58ca07
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522991"
---
# <a name="runaway-queries"></a>Consultas descontroladas

Una *consulta desejecución* es un tipo de error de [consulta parcial](partialqueryfailures.md) que se produce cuando se superó algún límite de [consulta](querylimits.md) interna durante la ejecución de la consulta.

Por ejemplo, se puede notificar el siguiente error:`HashJoin operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete.`

Hay varios cursos de acción posibles cuando esto sucede:
* Cambie la consulta para consumir menos recursos. Por ejemplo, si el error indica que el conjunto de resultados de la consulta es demasiado grande, puede intentar limitar el número de registros devueltos por la consulta (mediante el [operador take](../query/takeoperator.md) o agregando [cláusulas where](../query/whereoperator.md)adicionales), o reducir el número de columnas devueltas por la consulta (mediante el operador del [proyecto](../query/projectoperator.md) o el operador de salida del [proyecto),](../query/projectawayoperator.md)o usar el [operador de resumen](../query/summarizeoperator.md) para obtener datos agregados, etc.
* Aumente temporalmente el límite de consulta relevante para esa consulta (consulte **Memoria máxima por iterador** de conjunto de resultados en límites de [consulta](querylimits.md)).  
  Tenga en cuenta que esto no se recomienda en general, ya que existen los límites para proteger el clúster específicamente para asegurarse de que una sola consulta no interrumpa las consultas simultáneas que se ejecutan en el clúster.
  
  
  