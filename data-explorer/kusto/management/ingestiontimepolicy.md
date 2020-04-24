---
title: Directiva IngestionTime- Explorador de Azure Data Explorer ( IngestionTime) Microsoft Docs
description: En este artículo se describe la directiva IngestionTime en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 0c1755115a9e9f4c60c7f5574eb9b784520ecb88
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520832"
---
# <a name="ingestiontime-policy"></a>Directiva de IngestionTime

La directiva IngestionTime es una directiva opcional que se puede establecer (habilitar) en tablas.

Cuando esta directiva está habilitada en una tabla, `datetime` Kusto agrega `$IngestionTime`una columna oculta a la tabla, denominada . A partir de ese momento, cada vez que se ingenien nuevos datos en la tabla, el tiempo de ingestión (medido por el clúster de Kusto justo antes de que se confirmen los datos) se registra en la columna oculta para todos los registros que se ingiren. (Tenga en cuenta que `$IngestionTime` cada registro tiene su propio valor, al igual que cualquier otra columna.)

Como la columna de tiempo de ingesta está oculta, no se puede consultar directamente su valor.
En su lugar, se proporciona una función especial llamada [ingestion_time()](../query/ingestiontimefunction.md) para recuperar ese valor. Si no hay ninguna columna de este tipo en la tabla o la directiva IngestionTime no se ha habilitado cuando se ingirió un registro, se devuelve un valor nulo.

La directiva IngestionTime se ha diseñado para dos escenarios principales:
* Para permitir que los usuarios calculen la latencia de extremo a extremo en la ingesta de datos.
  Muchas tablas que contienen datos de registro tienen alguna columna de marca de tiempo cuyo valor se rellena con el origen para indicar la hora a la que se produjo el registro. Al comparar el valor de esa columna con la columna de tiempo de ingesta, se puede estimar la latencia para obtener los datos. (Tenga en cuenta que esto es sólo una estimación, porque la fuente y Kusto no necesariamente tienen sus relojes sincronizados.)
* Para admitir [cursores](../management/databasecursor.md)de base de datos, lo que permite a los usuarios emitir consultas consecutivas y cada vez limitar la consulta a los datos que se han ingerido desde la consulta anterior.



Para obtener más información sobre los comandos de control para administrar la directiva IngestionTime, [consulte aquí](../management/ingestiontime-policy.md).