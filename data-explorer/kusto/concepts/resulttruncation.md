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
ms.openlocfilehash: 41a9851405ff85210bba7728a3737fc3b0a57f70
ms.sourcegitcommit: fd3bf300811243fc6ae47a309e24027d50f67d7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83382393"
---
# <a name="query-result-set-has-exceeded-the-internal--limit"></a>El conjunto de resultados de la consulta ha superado el valor interno... ilimitado

Un *conjunto de resultados de la consulta ha superado el valor interno... Limit* es un tipo de [error de consulta parcial](partialqueryfailures.md) que se produce cuando el resultado de la consulta ha superado uno de dos límites:
* Límite en el número de registros ( `record count limit` , establecido de forma predeterminada en 500.000).
* Límite en la cantidad total de datos ( `data size limit` , que se establece de forma predeterminada en 67.108.864 (64 MB). 

Hay varios posibles cursos de acción cuando esto sucede:
* Cambie la consulta para consumir menos recursos. Por ejemplo, puede intentar limitar el número de registros devueltos por la consulta (mediante el [operador Take](../query/takeoperator.md) o agregando [cláusulas WHERE](../query/whereoperator.md)adicionales) o reducir el número de columnas devueltas por la consulta (mediante el operador de [proyecto](../query/projectoperator.md) o el [operador de proyecto](../query/projectawayoperator.md)), o usar el [operador de Resumen](../query/summarizeoperator.md) para obtener datos agregados, etc.
* Aumente el límite de consulta pertinente temporalmente para esa consulta (vea **truncamiento del resultado** en límites de la [consulta](querylimits.md)).  
  Tenga en cuenta que esto no se recomienda en general, dado que los límites existen para proteger el clúster específicamente para asegurarse de que una sola consulta no interrumpe las consultas simultáneas que se ejecutan en el clúster.