---
title: 'Tablas de hechos y dimensiones: Explorador de azure Data Explorer Microsoft Docs'
description: En este artículo se describen las tablas de hechos y dimensiones en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/23/2020
ms.openlocfilehash: 1a88f8549bf5accce197294d433ece8ccb683281
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523246"
---
# <a name="fact-and-dimension-tables"></a>Tablas de hechos y dimensiones

Al diseñar el esquema para una base de datos Kusto, es útil pensar en tablas como que pertenecen ampliamente a una de dos categorías:
* [Tablas de hechos](https://en.wikipedia.org/wiki/Fact_table) y
* [Tablas de dimensiones](https://en.wikipedia.org/wiki/Dimension_(data_warehouse)#Dimension_table).

**Las tablas de hechos** son tablas cuyos registros son "hechos" inmutables, como registros de servicio e información de mediciones. Los registros se anexan a la tabla progresivamente (de forma de streaming o en trozos grandes), y se mantienen allí hasta que tienen que ser eliminados por razones de costo o porque pierden su valor. De lo contrario, los registros nunca se actualizan.

**Las tablas** de dimensiones contienen datos de referencia (como tablas de búsqueda de un identificador de entidad a sus propiedades) y datos similares a instantáneas (tablas cuyo contenido completo cambia en una sola transacción). Por lo general, estas tablas no se ingestan regularmente con nuevos datos. En su lugar, todo el contenido de datos se actualiza a la vez mediante operaciones como [.set-or-replace](../management/data-ingestion/ingest-from-query.md), [.move extents](../management/extents-commands.md#move-extents)o [.rename tables](../management/rename-table-command.md).
En algunos casos, las tablas de dimensiones pueden derivarse de tablas de hechos a través de una directiva de [actualización](../management/updatepolicy.md) en la tabla de hechos, además de alguna consulta sobre la tabla que toma el último registro para cada entidad.

Es importante tener en cuenta que no hay manera de "marcar" una tabla en Kusto como una tabla de hechos o una tabla de dimensiones. Lo importante es cómo se ingentan los datos en la tabla y cómo se usan las tablas.

> [!NOTE]
> A veces, Kusto se utiliza para contener datos de entidad en tablas de hechos, de forma que los datos de la entidad cambian lentamente. Por ejemplo, datos relativos a alguna entidad física (por ejemplo, un equipo de oficina) que cambia de ubicación con poca frecuencia.
> Como los datos de Kusto son inmutables, la práctica común es`string`que cada tabla contiene dos columnas:`datetime`una columna de identidad ( ) que identifica la entidad y una última columna de marca de tiempo modificada ( ). A continuación, solo se recupera el último registro para cada identidad de entidad.



## <a name="commands-that-differentiate-fact-and-dimension-tables"></a>Comandos que diferencian las tablas de hechos y dimensiones

Hay procesos en Kusto que diferencian entre tablas de hechos y tablas de dimensiones.

* [exportación continua](../management/data-export/continuous-data-export.md)




Estos procesos están garantizados para procesar los datos de hecho de hecho una vez, confiando en el mecanismo de cursor de base de [datos.](../management/databasecursor.md)

Por ejemplo, cada ejecución de un trabajo de exportación continua exporta todos los registros que se han ingerido desde la última actualización del cursor de base de datos. Por lo tanto, los trabajos de exportación continua deben diferenciar entre las tablas de hechos (procesar solo los datos recién ingeridos) y las tablas de dimensiones (que se utilizan como búsquedas, por lo que se debe tener en cuenta toda la tabla).