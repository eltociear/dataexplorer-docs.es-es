---
title: Referencias de entidad- Explorador de Azure Data Explorer Microsoft Docs
description: En este artículo se describen las referencias de entidad en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: abbf63de632ff2ff5fb721dddc256c5ee62d4966
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81509408"
---
# <a name="entity-references"></a>Referencias a entidades

En la consulta se hace referencia a las entidades de esquema Kusto con nombre (bases de datos, tablas, columnas y funciones almacenadas, pero no clústeres) mediante sus nombres. Si el contenedor de la entidad es inequívoco en el contexto actual, el nombre de la entidad se puede usar sin cualificaciones adicionales. Por ejemplo, al ejecutar una `DB`consulta en una base `T` de datos denominada , `T`se puede hacer referencia a una tabla a la que se llama en esa base de datos simplemente con su nombre, .

Si, por el contrario, el contenedor de la entidad no está disponible desde el contexto, o si se desea hacer referencia a una entidad desde un contenedor distinto del contenedor en contexto, uno tiene que utilizar el **nombre completo**de la entidad, que es la concatenación del nombre de entidad al del contenedor (y potencialmente el de su contenedor, etc.) Por lo tanto, `DB` una consulta que `T1` se ejecuta `DB1` en una base `database("DB1").T1`de datos puede hacer referencia a una tabla de una base `cluster("https://C2.kusto.windows.net/").database("DB2").T2`de datos diferente del mismo clúster mediante , y si desea hacer referencia a una tabla de otro clúster, puede hacerlo, por ejemplo, mediante .

Las referencias de entidad también pueden el nombre bonito de la entidad, siempre que sea única en el contexto del contenedor de la entidad. Consulte [nombres bonitos](./entity-names.md#entity-pretty-names)de entidad .

## <a name="wildcard-matching-for-entity-names"></a>Coincidencia de caracteres comodín para nombres de entidad

En algunos contextos, se puede`*`usar un comodín ( ) para que coincida con todo o parte de un nombre de entidad. Por ejemplo, la siguiente consulta hace referencia a todas las `DB` tablas de `T`la base de datos actual, así como a todas las tablas de la base de datos cuyo nombre comienza con:

```kusto
union *, database("DB1").T*
```

Nota: La coincidencia de caracteres comodín no`$`coincide con los nombres de entidad que comienzan con un signo de dólar ( ).
Estos nombres están reservados por el sistema.



