---
title: Administración de permisos de base de datos de Azure Data Explorer
description: En este artículo se describen los controles de acceso basado en rol de bases de datos y tablas en el Explorador de datos de Azure.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/24/2018
ms.openlocfilehash: da56848dda4f129732408d062836586b1da4a0d3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492588"
---
# <a name="manage-azure-data-explorer-database-permissions"></a>Administración de permisos de base de datos del Explorador de datos de Azure

El Explorador de datos de Azure le permite controlar el acceso a bases de datos y tablas mediante un modelo de *control de acceso basado en rol*. Bajo este modelo, las *entidades de seguridad* (usuarios, grupos y aplicaciones) se asignan a *roles*. Las entidades de seguridad pueden acceder a recursos según los roles que tengan asignados.

En este artículo se describen los roles disponibles y cómo asignar entidades de seguridad a esos roles mediante Azure Portal y los comandos de administración del Explorador de datos de Azure.

## <a name="roles-and-permissions"></a>Roles y permisos

El Explorador de datos de Azure tiene los siguientes roles:

|Role                       |Permisos                                                                        |
|---------------------------|-----------------------------------------------------------------------------------|
|Administrador de base de datos             |Puede hacer cualquier cosa en el ámbito de una base de datos determinada.|
|Usuario de la base de datos              |Puede leer todos los datos y metadatos de la base de datos. Además, puede crear tablas (y convertirse en el administrador de tablas de esa tabla) y funciones en la base de datos.|
|Visor de base de datos            |Puede leer todos los datos y metadatos de la base de datos.|
|Agente de ingesta de base de datos          |Puede ingerir datos de todas las tablas existentes de la base de datos, pero no consultar los datos.|
|Supervisor de base de datos           |Puede ejecutar comandos ".show ..." en el contexto de la base de datos y sus entidades secundarias.|
|Administrador de tablas                |Puede hacer cualquier cosa en el ámbito de una tabla determinada. |
|Agente de ingesta de tablas             |Puede ingerir datos en el ámbito de una determinada tabla, pero no consultar los datos.|

## <a name="manage-permissions-in-the-azure-portal"></a>Administración de permisos en Azure Portal

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).

1. Vaya a su clúster del Explorador de datos de Azure.

1. En la sección **Información general**, seleccione la base de datos donde quiere administrar permisos.

    ![Seleccionar base de datos](media/manage-database-permissions/select-database.png)

1. Seleccione **Permisos** y luego **Agregar**.

    ![Permisos de base de datos](media/manage-database-permissions/database-permissions.png)

1. En **Add database permissions** (Agregar permisos de base de datos), seleccione el rol que quiere asignar a la entidad y, luego, **Select principals** (Seleccionar entidades de seguridad).

    ![Incorporación de permisos de base de datos](media/manage-database-permissions/add-permission.png)

1. Busque la entidad, selecciónela y, luego, elija **Seleccionar**.

    ![Administración de permisos en Azure Portal](media/manage-database-permissions/new-principals.png)

1. Seleccione **Guardar**.

    ![Administración de permisos en Azure Portal](media/manage-database-permissions/save-permission.png)

## <a name="manage-permissions-with-management-commands"></a>Administración de permisos con comandos de administración

1. Inicie sesión en [https://dataexplorer.azure.com](https://dataexplorer.azure.com) y agregue el clúster si aún no está disponible.

1. En el panel izquierdo, seleccione la base de datos adecuada.

1. Use el comando `.add` para asignar roles a entidades: `.add database databasename rolename ('aaduser | aadgroup=user@domain.com')`. Para agregar el rol de usuario a un usuario de base de datos, ejecute el siguiente comando y sustituya el nombre y el usuario de base de datos.

    ```Kusto
    .add database <TestDatabase> users ('aaduser=<user@contoso.com>')
    ```

    La salida del comando muestra la lista de usuarios existentes y los roles que tienen asignados en la base de datos.
    
    Para ver ejemplos relacionados con Azure Active Directory y el modelo de autorización de Kusto, consulte el artículo de [principios y proveedores de identidades](kusto/management/access-control/principals-and-identity-providers.md).

## <a name="next-steps"></a>Pasos siguientes

[Escritura de consultas](write-queries.md)
