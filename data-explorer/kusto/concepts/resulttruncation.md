---
title: El conjunto de resultados de la consulta ha superado el ... Límite- Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describe que el conjunto de resultados de consulta ha superado el ... límite en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/23/2018
ms.openlocfilehash: efb07e68922a88e2cf59ef1eefeabd3af085aed4
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523025"
---
# <a name="query-result-set-has-exceeded-the-internal--limit"></a>El conjunto de resultados de la consulta ha superado el ... Límite

Un conjunto de resultados de *consulta ha superado el ... límite* es un tipo de error de [consulta parcial](partialqueryfailures.md) que se produce cuando el resultado de la consulta ha superado uno de los dos límites:
* Límite en el número`record count limit`de registros ( , establecido de forma predeterminada en 500.000).
* Límite de la cantidad total`data size limit`de datos ( , establecido de forma predeterminada en 67.108.864 (64MB)). 

Hay varios cursos de acción posibles cuando esto sucede:
* Cambie la consulta para consumir menos recursos. Por ejemplo, puede intentar limitar el número de registros devueltos por la consulta (mediante el [operador take](../query/takeoperator.md) o agregando [cláusulas where](../query/whereoperator.md)adicionales), o reducir el número de columnas devueltas por la consulta (mediante el operador del [proyecto](../query/projectoperator.md) o el operador de salida del [proyecto),](../query/projectawayoperator.md)o usar el [operador de resumen](../query/summarizeoperator.md) para obtener datos agregados, etc.
* Aumente temporalmente el límite de consulta relevante para esa consulta (consulte **Truncamiento** de resultados en límites de [consulta).](querylimits.md)  
  Tenga en cuenta que esto no se recomienda en general, ya que existen los límites para proteger el clúster específicamente para asegurarse de que una sola consulta no interrumpa las consultas simultáneas que se ejecutan en el clúster.