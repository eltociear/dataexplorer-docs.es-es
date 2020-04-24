---
title: 'Prácticas recomendadas para el diseño de esquemas: Explorador de azure Data Explorer Microsoft Docs'
description: En este artículo se describen los procedimientos recomendados para el diseño de esquemas en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/18/2020
ms.openlocfilehash: f9e2d4a2ad50b140d041179736b48a41a424f5c6
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522175"
---
# <a name="best-practices-for-schema-design"></a>Prácticas recomendadas para el diseño de esquemas

Hay varios "Dos y No?" que puede seguir para que sus comandos de administración funcionen mejor y tengan un efecto más ligero en el servicio.

## <a name="do"></a>Lo que es necesario hacer:

1. Si necesita crear varias tablas, utilice el [`.create tables`](create-tables-command.md) comando, en lugar de emitir muchos `.create table` comandos.
2. Si necesita cambiar el nombre de varias tablas, haga esto con una sola llamada a [`.rename tables`](rename-table-command.md), en lugar de emitir una llamada independiente para cada par de tablas.
3. Utilice el `.show` comando de ámbito más bajo, en`|`lugar de aplicar filtros después de una tubería ( ). Por ejemplo:
    - Usar `.show table T extents` en lugar de `.show cluster extents | where TableName == 'T'`
    - Use `.show database DB schema` en lugar de `.show schema | where DatabaseName == 'DB'`.
4. Utilícelo `.show table T` solo si necesita obtener estadísticas reales en una sola tabla. Si solo necesita comprobar la existencia de la tabla, o `.show table T schema as json`simplemente obtener el esquema de la tabla, utilice .
5. Al definir el esquema de una tabla que incluirá valores datetime, asegúrese de que estas columnas se escriben con el `datetime` tipo.
    - Kusto está altamente optimizado `datetime` para filtrar columnas. No convierta `string` columnas o numéricas `long`(por `datetime` ejemplo, ) en el momento de la consulta para filtrar, si eso se puede hacer antes o durante el tiempo de ingesta.

## <a name="dont"></a>Lo que debe evitar:

1. No `.show` ejecute comandos con demasiada frecuencia `.show schema`(por ejemplo, , `.show databases`, `.show tables`). Cuando sea posible: almacene en caché la información que devuelven.
2. No ejecute `.show schema` el comando en un clúster que sea un esquema grande (por ejemplo, con más de 100 bases de datos). En su [`.show databases schema`](../management/show-schema-database.md)lugar, utilice .
3. No ejecute operaciones [de comando y luego consulta](index.md#combining-queries-and-control-commands) con demasiada frecuencia.
    - *comando-then-query*: significa canalizar el conjunto de resultados del comando de control y aplicar filtros/agregaciones en él.
        - Por ejemplo: `.show ... | where ... | summarize ...`
    - Al ejecutar algo `.show cluster extents | count` como: `| count`(énfasis en el ), Kusto primero prepara una tabla de datos que contiene todos los detalles sobre todas las extensiones en el clúster y, a continuación, envía esa tabla solo en memoria al motor de Kusto para hacer el recuento. Esto significa que Kusto realmente trabaja muy duro en un camino sin optimizada para devolverte una respuesta tan trivial.
4. Uso excesivo de etiquetas de extensión como parte de la ingesta de datos. Especialmente cuando `drop-by:` se utilizan etiquetas, que limitan la capacidad del sistema para realizar procesos de aseo orientados al rendimiento en segundo plano.
    - Vea las notas de rendimiento [aquí](../management/extents-overview.md#extent-tagging).
    