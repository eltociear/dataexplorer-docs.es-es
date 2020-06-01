---
title: 'Directiva de IngestionTime: Azure Explorador de datos'
description: En este artículo se describe la Directiva IngestionTime en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 50e0083b1cdbed06106507fe69fb0d039c923c43
ms.sourcegitcommit: 9fe6e34ef3321390ee4e366819ebc9b132b3e03f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/01/2020
ms.locfileid: "84257814"
---
# <a name="ingestiontime-policy"></a>Directiva de IngestionTime

La Directiva IngestionTime es una directiva opcional que se puede establecer (habilitada) en las tablas.

Cuando está habilitada, Kusto agrega una `datetime` columna oculta a la tabla, denominada `$IngestionTime` . Ahora, cada vez que se ingestan nuevos datos, el tiempo de ingesta se registra en la columna oculto. Ese tiempo lo mide el clúster de Kusto justo antes de que se confirmen los datos. 

> [!NOTE]
> Cada registro tiene su propio `$IngestionTime` valor.

Dado que la columna de tiempo de ingesta está oculta, no puede consultar directamente su valor.
En su lugar, una función especial llamada [ingestion_time ()](../query/ingestiontimefunction.md) recupera ese valor. Si no hay ninguna `datetime` columna en la tabla o no se ha habilitado la Directiva IngestionTime cuando se ingesta un registro, se devuelve un valor null.

La Directiva IngestionTime está diseñada para dos escenarios principales:
* Permite a los usuarios calcular la latencia de la ingesta de datos.
  Muchas tablas con datos de registro tienen una columna de marca de tiempo. El valor de marca de tiempo lo rellena el origen e indica la hora a la que se generó el registro. Al comparar el valor de la columna con la columna de tiempo de ingesta, puede calcular la latencia para obtener los datos. 
  
  > [!NOTE]
  > El valor calculado es solo una estimación, ya que el origen y Kusto no tienen necesariamente sincronizados sus relojes.
  
* Para admitir [cursores de base](../management/databasecursor.md) de datos que permitan a los usuarios emitir consultas consecutivas, la consulta se limita a los datos que se han ingerido desde la consulta anterior.

Para obtener más información, Vea los [comandos de control para administrar la Directiva IngestionTime](../management/ingestiontime-policy.md).
