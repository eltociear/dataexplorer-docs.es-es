---
title: Tarea de Azure DevOps para Azure Data Explorer
description: En este tema, aprenderá a crear canalizaciones de versión e implementaciones.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: jasonh
ms.service: data-explorer
ms.topic: conceptual
ms.date: 05/05/2019
ms.openlocfilehash: cf6724a0f363eda510b5aa9cde195ec4fd3db65b
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373867"
---
# <a name="azure-devops-task-for-azure-data-explorer"></a>Tarea de Azure DevOps para Azure Data Explorer

[Azure DevOps Services](https://azure.microsoft.com/services/devops/) proporciona herramientas de colaboración para el desarrollo, como canalizaciones de alto rendimiento, repositorios de Git privados gratuitos, paneles Kanban configurables y amplias capacidades de pruebas automatizadas y continuas. [Azure Pipelines](https://azure.microsoft.com/services/devops/pipelines/) es una capacidad de Azure DevOps que permite administrar CI/CD para implementar el código con las canalizaciones de alto rendimiento que funcionan con cualquier lenguaje, plataforma y nube.
[Azure Data Explorer - Admin Commands](https://marketplace.visualstudio.com/items?itemName=Azure-Kusto.PublishToADX) es la tarea de Azure Pipelines que permite crear canalizaciones de versión e implementar los cambios de base de datos en sus bases de datos de Azure Data Explorer. Está disponible de forma gratuita en [Visual Studio Marketplace](https://marketplace.visualstudio.com/).

En este documento se describe un ejemplo sencillo de uso de la tarea **Azure Data Explorer – Admin Commands** para implementar los cambios de esquema en su base de datos. Para obtener información sobre las canalizaciones de CI/CD completas, consulte la [documentación de Azure DevOps](/azure/devops/user-guide/what-is-azure-devops?view=azure-devops#vsts).

## <a name="prerequisites"></a>Prerrequisitos

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* Instalación del clúster de Azure Data Explorer:
    * Un [clúster y una base de datos de Azure Data Explorer](create-cluster-database-portal.md).
    * Cree aplicaciones de Azure Active Directory (Azure AD) mediante el [aprovisionamiento de una aplicación de Azure AD](kusto/management/access-control/how-to-provision-aad-app.md).
    * Conceda acceso a su aplicación de Azure AD en la base de datos de administrando [permisos de base de datos de Azure Data Explorer](manage-database-permissions.md).
* Instalación de Azure DevOps:
    * Registro en una [organización gratuita](/azure/devops/user-guide/sign-up-invite-teammates?view=azure-devops)
    * [Creación de una organización](/azure/devops/organizations/accounts/create-organization?view=azure-devops)
    * [Creación de un proyecto en Azure DevOps](/azure/devops/organizations/projects/create-project?view=azure-devops)
    * [Código con Git](/azure/devops/user-guide/code-with-git?view=azure-devops)

## <a name="create-folders"></a>Crear carpetas

Cree las siguientes carpetas de ejemplo (*Funciones*, *Directivas* y *Tablas*) en el repositorio de Git. Copie los archivos de [aquí](https://github.com/Azure/azure-kusto-docs-samples/tree/master/DevOps_release_pipeline) en las carpetas respectivas, como se muestra a continuación, y confirme los cambios. Los archivos de ejemplo se proporcionan para ejecutar el siguiente flujo de trabajo.

![Crear carpetas](media/devops/create-folders.png)

> [!TIP]
> Al crear su propio flujo de trabajo, le recomendamos que haga su código idempotente. Por ejemplo, use [.create-merge table](kusto/management/create-table-command.md#create-merge-table) en lugar de [.create table](kusto/management/create-table-command.md) y use la función [.create-or-alter](kusto/management/create-alter-function.md) en lugar de la función [.create](kusto/management/create-function.md).

## <a name="create-a-release-pipeline"></a>Creación de una canalización de versión

1. Inicie sesión en su [organización de Azure DevOps](https://dev.azure.com/).
1. Seleccione **Canalizaciones** > **Versiones** en el menú izquierdo y elija **Nueva canalización**.

    ![Nueva canalización](media/devops/new-pipeline.png)

1. Se abrirá la ventana **Nueva canalización de versión**. En la pestaña **Canalizaciones**, en el panel **Seleccionar una plantilla**, elija **Empty job** (Trabajo vacío).

     ![Seleccione una plantilla:](media/devops/select-template.png)

1. Seleccione el botón **Fase**. En el panel **Fase**, agregue el valor de **Nombre de la fase**. Seleccione **Guardar** para guardar la canalización.

    ![Nombre de la fase](media/devops/stage-name.png)

1. Seleccione el botón **Agregar un artefacto**. En el panel **Agregar un artefacto**, seleccione el repositorio donde existe el código, rellene la información pertinente y haga clic en **Agregar**. Seleccione **Guardar** para guardar la canalización.

    ![Agregar un artefacto](media/devops/add-artifact.png)

1. En la pestaña **Variables**, seleccione **+ Agregar** para crear una variable para **Dirección URL de extremo** que se usará en la tarea. Escriba los valores de **Nombre** y **Valor** del punto de conexión. Seleccione **Guardar** para guardar la canalización. 

    ![Creación de variables](media/devops/create-variable.png)

    Para buscar su Endpoint_URL, la página de información general del **clúster de Azure Data Explorer** en Azure Portal contiene el URI del clúster de Azure Data Explorer. Cree el URI con el siguiente formato `https://<Azure Data Explorer cluster URI>?DatabaseName=<DBName>`.  Por ejemplo, https:\//kustodocs.westus.kusto.windows.net?DatabaseName=SampleDB.

    ![URI del clúster de Azure Data Explorer](media/devops/adx-cluster-uri.png)

## <a name="create-tasks-to-deploy"></a>Creación de tareas para implementar

1. En la pestaña **Canalización**, haga clic en **1 job, 0 task** (1 trabajo, 0 tareas) para agregar tareas. 

    ![Adición de tareas](media/devops/add-task.png)

1. Cree tres tareas para implementar **Tablas**, **Funciones** y **Directivas** en este orden. 

1. En la pestaña **Tareas**, seleccione **+** por **Trabajo de agente**. Busque **Azure Data Explorer**. En **Marketplace**, instale la extensión **Azure Data Explorer – Admin Commands**. A continuación, seleccione **Agregar** en **Run Azure Data Explorer Command** (Ejecutar comando de Azure Data Explorer).

     ![Adición de comandos de administrador](media/devops/add-admin-commands.png)

1. Haga clic en **Kusto Command** (Comando Kusto), situado a la izquierda, y actualice la tarea con la siguiente información:
    * **Nombre para mostrar**: nombre de la tarea.
    * **Ruta de acceso del archivo**: en la tarea **Tablas**, especifique */Tables/* .csl, ya que los archivos de creación de tabla se encuentran en la carpeta *Tabla*.
    * **Dirección URL de extremo**: escriba la variable `EndPoint URL` creada en el paso anterior.
    * Seleccione **Use Service Endpoint** (Usar punto de conexión de servicio) y elija **+ Nuevo**.

    ![Actualización de tarea del comando Kusto](media/devops/kusto-command-task.png)

1. Complete la siguiente información en la ventana **Add Azure Data Explorer service connection** (Agregar conexión del servicio de Azure Data Explorer):

    |Configuración  |Valor sugerido  |
    |---------|---------|
    |**Nombre de la conexión**     |    Escriba un nombre para identificar este punto de conexión de servicio.     |
    |**URL de clúster**    |    El valor se puede encontrar en la sección de información general del clúster de Azure Data Explorer en Azure Portal. | 
    |**Id. de entidad de servicio**    |    Escriba el Id. de aplicación de AAD (creado como requisito previo).     |
    |**Service Principal App Key** (Clave de la aplicación de entidad de servicio)     |    Escriba la clave de la aplicación de AAD (creada como requisito previo).    |
    |**Id. de inquilino de AAD**    |      Escriba el inquilino de AAD (por ejemplo, microsoft.com, contoso.com, etc.).    |

    Seleccione la casilla **Permita que todas las canalizaciones usen esta conexión**. Seleccione **Aceptar**.

    ![Adición de conexión a servicios](media/devops/add-service-connection.png)

1. Repita los pasos del 1 al 5 dos veces más para implementar archivos de las carpetas *Funciones* y *Directivas*. Seleccione **Guardar**. En la pestaña **Tareas**, vea las tres tareas creadas: **Deploy Tables** (Implementar tablas), **Deploy Functions** (Implementar funciones) y **Deploy Policies** (Implementar directivas).

    ![Implementación de todas las carpetas](media/devops/deploy-all-folders.png)

1. Seleccione **+Versión** > **Crear versión** para crear una versión.

    ![Creación de una versión](media/devops/create-release.png)

1. En la pestaña **Registros**, compruebe si el estado de implementación es correcto.

    ![La implementación se ha realizado correctamente](media/devops/deployment-successful.png)

Ahora ha completado la creación de una canalización de versión para la implementación de tres tareas en preproducción.