---
title: 'Consultas descontroladas: Azure Explorador de datos'
description: En este artículo se describen las consultas descontroladas en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: 8300289cb98e2f23613c15a711982efe06f327cf
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84780259"
---
# <a name="runaway-queries"></a>Consultas descontroladas

Una *consulta descontrolada* es un tipo de [error de consulta parcial](partialqueryfailures.md) que se produce cuando se supera algún [límite de consulta](querylimits.md) interno durante la ejecución de la consulta. 

Por ejemplo, puede aparecer el siguiente error:`HashJoin operator has exceeded the memory budget during evaluation. Results may be incorrect or incomplete.`

Hay varios posibles cursos de acción.
* Cambie la consulta para consumir menos recursos. Por ejemplo, si el error indica que el conjunto de resultados de la consulta es demasiado grande, puede:
  * Limitar el número de registros devueltos por la consulta
     * Usar el [operador Take](../query/takeoperator.md)
     * Agregar [cláusulas WHERE](../query/whereoperator.md) adicionales
  * Reducir el número de columnas que devuelve la consulta 
     * Usar el [operador de proyecto](../query/projectoperator.md)
     * Uso del [operador de ubicación de proyecto](../query/projectawayoperator.md)
  * Use el [operador de Resumen](../query/summarizeoperator.md) para obtener datos agregados.
* Aumente el límite de consulta pertinente temporalmente para esa consulta. Para obtener más información, consulte [límites de consultas: límite de memoria por iterador](querylimits.md). Sin embargo, no se recomienda este método. Los límites existen para proteger el clúster y para asegurarse de que una sola consulta no interrumpa las consultas simultáneas que se ejecutan en el clúster.
