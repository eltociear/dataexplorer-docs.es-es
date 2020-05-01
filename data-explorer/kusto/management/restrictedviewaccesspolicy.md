---
title: 'Consultas de controles de directiva de Kusto RestrictedViewAccess: Azure Explorador de datos'
description: En este artículo se describe la Directiva RestrictedViewAccess en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: e44aa2b14aa8babdab95a6ad8c6f7ef5ed026ff9
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617433"
---
# <a name="restrictedviewaccess-policy"></a>Directiva RestrictedViewAccess

*RestrictedViewAccess* es una directiva opcional que se puede habilitar en las tablas de una base de datos.

Cuando esta directiva está habilitada en una tabla, los datos de la tabla *solo* se pueden consultar a las entidades de seguridad agregadas al rol [UnrestrictedViewer](../management/access-control/role-based-authorization.md) en la base de datos.

Cuando se habilita una directiva en una tabla, cualquier entidad de seguridad (incluso una tabla, una base de datos o un administrador de clústeres) que no se incluye en el rol de nivel de base de datos [UnrestrictedViewer](../management/access-control/role-based-authorization.md) , no podrá consultar los datos de la tabla.

El rol [UnrestrictedViewer](../management/access-control/role-based-authorization.md) concede el permiso View a *todas* las tablas de la base de datos que tienen habilitada la Directiva, suponiendo que la entidad de seguridad actual ya está autorizada para consultar la base de datos (un administrador de base de datos/usuario/visor). La adición o eliminación de entidades de seguridad a o desde el rol puede realizarse mediante un [DatabaseAdmin](../management/access-control/role-based-authorization.md).

> [!NOTE]
> No se puede configurar la Directiva de RestrictedViewAccess en una tabla en la que está habilitada una [Directiva de seguridad de nivel de fila](./rowlevelsecuritypolicy.md) .

Para obtener más información sobre los comandos de control para administrar la Directiva RestrictedViewAccess, [Consulte aquí](../management/restrictedviewaccess-policy.md).