---
title: 'Entidades: Azure Data Explorer | Microsoft Docs'
description: En este artículo se describen las entidades de Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 11/19/2019
ms.openlocfilehash: a69362b590acee99fbe9b57d9303099f0033d458
ms.sourcegitcommit: 4f576c1b89513a9e16641800abd80a02faa0da1c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/22/2020
ms.locfileid: "85128502"
---
# <a name="entity-types"></a>Tipos de entidades

Las consultas de Kusto se ejecutan en el contexto de una base de datos de Kusto que está asociada a un clúster de Kusto. Los datos de la base de datos se organizan en tablas a las que puede hacer referencia la consulta y, dentro de la tabla, se organizan en forma de cuadrícula rectangular de columnas y filas. Además, las consultas pueden hacer referencia a funciones almacenadas en la base de datos, que son fragmentos de consulta disponibles para su reutilización.

* Los clústeres son entidades que contienen bases de datos.
  Los clústeres no tienen nombre, pero se puede hacer referencia a ellos mediante la función especial `cluster()` con el identificador URI del clúster.
  Por ejemplo, `cluster("https://help.kusto.windows.net")` es una referencia a un clúster que contiene la base de datos `Samples`.

* Las [bases de datos](./databases.md) son entidades con nombre que contienen tablas y funciones almacenadas. Todas las consultas de Kusto se ejecutan en el contexto de una base de datos y la consulta puede hacer referencia a las entidades de esa base de datos sin cualificaciones. Además, se puede hacer referencia a otras bases de datos del clúster, o a bases de datos de otros clústeres, mediante la [función especial database()](../databasefunction.md). Por ejemplo: `cluster("https://help.kusto.windows.net").database("Samples")`
  es una referencia universal a una base de datos concreta.

* Las [tablas](./tables.md) son entidades con nombre que contienen datos. Una tabla tiene un conjunto ordenado de columnas y cero o más filas de datos, cada una de las cuales contiene un valor de datos para cada una de las columnas de la tabla. Solo se puede hacer referencia a las tablas por nombre si están en la base de datos en el contexto de la consulta o si se califican con una referencia de base de datos en caso contrario. Por ejemplo, `cluster("https://help.kusto.windows.net").database("Samples").StormEvents` es una referencia universal a una tabla concreta de la base de datos `Samples`.
  También se puede hacer referencia a las tablas mediante la [función especial table()](../tablefunction.md).

* Las [columnas](./columns.md) son entidades con nombre que [tienen un tipo de datos escalar](../scalar-data-types/index.md).
  La referencia a las columnas en la consulta se hace con relación al flujo de TDS que se encuentra en el contexto del operador concreto que hace referencia a ellas.

* Las [funciones almacenadas](./stored-functions.md) son entidades con nombre que permiten reutilizar las consultas de Kusto o los elementos de las consultas.

* Las [tablas externas](./externaltables.md) son entidades que hacen referencia a datos almacenados fuera de la base de datos de Kusto.
  Las tablas externas se usan para exportar datos de Kusto al almacenamiento externo, así como para consultar datos externos sin necesidad de ingerirlos en Kusto.