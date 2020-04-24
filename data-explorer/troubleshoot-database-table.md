---
title: Error al crear o eliminar una base de datos o una tabla en Azure Data Explorer
description: En este artículo se describen los pasos para solucionar problemas de creación de bases de datos y tablas en el Explorador de datos de Azure.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: 67e4c33498b05884fe667a7977ddb40e647ab4c8
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493584"
---
# <a name="troubleshoot-failure-to-create-or-delete-a-database-or-table-in-azure-data-explorer"></a>Solución de problemas: Error al crear o eliminar una base de datos o una tabla en el Explorador de datos de Azure

En el Explorador de datos de Azure, normalmente se trabaja con bases de datos y tablas. En este artículo se proporcionan también los pasos para la solución de problemas que puedan surgir.

## <a name="creating-a-database"></a>Creación de una base de datos

1. Asegúrese de que tiene los permisos adecuados. Para crear una base de datos, debe ser miembro del rol *colaborador* o *propietario* en la suscripción de Azure. Si es necesario, hable con su administrador de suscripciones para que pueda agregarle el rol adecuado.

1. Asegúrese de que el nombre de la base de datos no tenga errores de validación de nombres. El nombre debe ser alfanumérico, con una longitud máxima de 260 caracteres.

1. Asegúrese de que los valores de período de retención de la base de datos y almacenamiento en caché estén dentro de los intervalos permitidos. La retención debe estar comprendida entre 1 y 36500 días (100 años). El almacenamiento en caché debe estar entre 1 y el valor máximo establecido para la retención.

## <a name="deleting-or-renaming-a-database"></a>Eliminación o cambio de nombre de una base de datos

Asegúrese de que tiene los permisos adecuados. Para eliminar una base de datos o cambiarle el nombre, debe ser miembro del rol *colaborador* o *propietario* en la suscripción de Azure. Si es necesario, hable con su administrador de suscripciones para que pueda agregarle el rol adecuado.

## <a name="creating-a-table"></a>Creación de una tabla

1. Asegúrese de que tiene los permisos adecuados. Para crear una tabla, debe ser miembro del rol *administrador de base de datos* o *usuario de base de datos* en la base de datos, o del rol *colaborador* o *propietario* de la suscripción de Azure. Si es necesario, hable con su administrador de suscripciones o clústeres para que pueda agregarle el rol adecuado.

    Para más información acerca de los permisos, consulte [Administración de permisos de bases de datos](manage-database-permissions.md).

1. Asegúrese de que no haya una tabla con el mismo nombre. Si la hay, puede: crear una tabla con un nombre diferente; cambiar el nombre de la tabla existente (requiere el rol *administrador de tablas*); o quite la tabla existente (requiere el rol *administrador de base de datos*). Use los comandos siguientes.

    ```Kusto
    .drop table <TableName>

   .rename table <OldTableName> to <NewTableName>
    ```

## <a name="deleting-or-renaming-a-table"></a>Eliminación o cambio de nombre de una tabla

Asegúrese de que tiene los permisos adecuados. Para eliminar o cambiar el nombre de una tabla, debe ser miembro de la *Administrador de base de datos* o *tabla admin* rol en la base de datos. Si es necesario, hable con su administrador de suscripciones o clústeres para que pueda agregarle el rol adecuado.

Para más información acerca de los permisos, consulte [Administración de permisos de bases de datos](manage-database-permissions.md).

## <a name="general-guidance"></a>Instrucciones generales

1. Compruebe el [panel de mantenimiento de los servicios de Azure](https://azure.microsoft.com/status/). Busque el estado del Explorador de datos de Azure en la región donde está intentando trabajar con una base de datos o tabla.

    Si el estado no es **Correcto** (marca de verificación verde), pruebe otra vez después de que el estado mejore.

1. Si aún necesita ayuda para solucionar el problema, abra una solicitud de soporte técnico en [Azure Portal](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview).

## <a name="next-steps"></a>Pasos siguientes

[Comprobación del estado del clúster](check-cluster-health.md)