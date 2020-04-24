---
title: 'Directiva RestrictedViewAccess: Explorador de azure Data Explorer ( Microsoft Docs'
description: En este artículo se describe la directiva RestrictedViewAccess en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 6f994f5b80632650ab6dbe5dcf28cd82407d688f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520424"
---
# <a name="restrictedviewaccess-policy"></a>Directiva RestrictedViewAccess

*RestrictedViewAccess* es una directiva opcional que se puede habilitar en las tablas de una base de datos.

Cuando esta directiva está habilitada en una tabla, los datos de la tabla *solo* se pueden consultar a las entidades de seguridad agregadas al rol [UnrestrictedViewer](../management/access-control/role-based-authorization.md) de la base de datos.

Cuando se habilita una directiva en una tabla, cualquier entidad de seguridad (incluso una tabla/base de datos/administrador de clúster) que no se incluye en el rol de nivel de base de datos [UnrestrictedViewer,](../management/access-control/role-based-authorization.md) no podrá consultar datos en la tabla.

El rol [UnrestrictedViewer](../management/access-control/role-based-authorization.md) concede permiso de vista a *todas las* tablas de la base de datos que tienen habilitada la directiva, suponiendo que la entidad de seguridad actual ya está autorizada para consultar la base de datos (un administrador/usuario/espectador de la base de datos). Agregar o quitar entidades de seguridad al rol puede realizarlo un [Administrador de bases](../management/access-control/role-based-authorization.md)de datos .

> [!NOTE]
> La directiva RestrictedViewAccess no se puede configurar en una tabla en la que está habilitada una directiva de seguridad de nivel de [fila.](./rowlevelsecuritypolicy.md)

Para obtener más información sobre los comandos de control para administrar la directiva RestrictedViewAccess, [consulte aquí](../management/restrictedviewaccess-policy.md).