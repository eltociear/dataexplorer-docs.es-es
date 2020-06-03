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
ms.openlocfilehash: 9ed8ba6ac5e66326e6aa1d9e7a7f474506b57e26
ms.sourcegitcommit: f7101c6b41ec250d05f4cb6092e2939958b37b40
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2020
ms.locfileid: "84328983"
---
# <a name="cross-cluster-queries-and-schema-changes"></a>Consultas entre clústeres y cambios de esquema

Al ejecutar una consulta entre clústeres, el clúster que realiza la interpretación de la consulta inicial debe tener el esquema de las entidades a las que se hace referencia en los clústeres remotos.
Cuando se envía la siguiente consulta a *Cluster1*

```kusto
Table1 | join ( cluster("Cluster2").database("SomeDB").Table2 ) on KeyColumn
``` 

*Cluster1* debe saber las columnas que tiene la *Tabla2* en *Cluster2* y los tipos de datos de esas columnas. Para obtener esa información, el comando especial se envía desde *Cluster1* a *Cluster2*.
El envío del comando puede ser caro. Una vez recuperados, los esquemas para las entidades remotas se almacenan en caché y la consulta siguiente que hace referencia a las mismas entidades se ejecutará menos operaciones de red.

Los cambios en el esquema de la entidad remota pueden producir efectos no deseados. Por ejemplo, las columnas agregadas no se reconocen o las columnas eliminadas causan un error de consulta parcial en lugar de un error semántico.

Para solucionar este problema, los esquemas almacenados en caché expiran una hora después de la recuperación, de modo que cualquier consulta se ejecute más de una hora después del cambio, funcionará con el esquema actualizado.
