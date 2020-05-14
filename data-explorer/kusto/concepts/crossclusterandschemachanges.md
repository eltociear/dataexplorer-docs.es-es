---
title: 'Kusto de consultas entre clústeres & cambios de esquema: Azure Explorador de datos'
description: En este artículo se describen las consultas entre clústeres y los cambios de esquema en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 58fd8cef3836d17eb327734338b1567d815d7349
ms.sourcegitcommit: fd3bf300811243fc6ae47a309e24027d50f67d7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83382036"
---
# <a name="cross-cluster-queries-and-schema-changes"></a>Consultas entre clústeres y cambios de esquema 

Al ejecutar una consulta entre clústeres, el clúster que realiza la interpretación inicial de la consulta debe tener el esquema de las entidades a las que se hace referencia en los clústeres remotos.
Por lo tanto, cuando la siguiente consulta se envía a *Cluster1*

```kusto
Table1 | join ( cluster("Cluster2").database("SomeDB").Table2 ) on KeyColumn
``` 

*Cluster1* debe saber qué columnas de *Cluster2* tiene la *Tabla2* y cuáles son los tipos de datos de esas columnas. Para lograr que el comando especial se envíe desde *Cluster1* a *Cluster2*.
Este envío de este comando puede ser bastante costoso, así que, una vez recuperados, los esquemas de las entidades remotas se almacenan en caché y la siguiente consulta que hace referencia a las mismas entidades tendrá que ejecutar menos operaciones de red.

Esto puede producir efectos no deseados cuando cambia el esquema de la entidad remota (se han agregado columnas que no se reconocen o eliminan columnas que causan un error de consulta parcial en lugar de un error semántico).

Para mitigar este problema, los esquemas almacenados en caché expiran en 1 hora después de la recuperación, por lo que cualquier consulta que se ejecute en más de una hora después del cambio funcionará con el esquema actualizado.