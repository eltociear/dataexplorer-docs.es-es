---
title: 'Consultas entre clústeres y cambios de esquema: Explorador de azure Data Explorer Microsoft Docs'
description: En este artículo se describen las consultas entre clústeres y los cambios de esquema en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a9473add657fe8a888818f83c72f12d47138c00e
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523297"
---
# <a name="cross-cluster-queries-and-schema-changes"></a>Consultas entre clústeres y cambios de esquema 

Al ejecutar la consulta entre clústeres, el clúster que realiza la interpretación de la consulta inicial debe tener el esquema de las entidades a las que se hace referencia en clústeres remotos.
Así que cuando la siguiente consulta enviada a *Cluster1*

```kusto
Table1 | join ( cluster("Cluster2").database("SomeDB").Table2 ) on KeyColumn
``` 

*Cluster1* debe saber qué columnas tiene *Table2* en *Cluster2* y cuáles son los tipos de datos de esas columnas. Para lograr que el comando especial se envíe de *Cluster1* a *Cluster2.*
Este envío de este comando puede ser bastante costoso, por lo que una vez recuperado, los esquemas para entidades remotas se almacenan en caché y la siguiente consulta que hace referencia a las mismas entidades tendrá que ejecutar menos operaciones de red.

Esto puede provocar efectos no deseados cuando cambia el esquema de la entidad remota (columnas agregadas que no se reconocen o columnas eliminadas que provocan 'Error de consulta parcial' en lugar de error semántico).

Para aliviar este problema, los esquemas almacenados en caché expiran en 1 hora después de la recuperación, por lo que cualquier consulta ejecutada en más de 1 hora después del cambio funcionará con el esquema actualizado.