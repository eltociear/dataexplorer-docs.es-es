---
author: orspod
ms.service: data-explorer
ms.topic: include
ms.date: 11/03/2019
ms.author: orspodek
ms.openlocfilehash: 3cd9d017429b629acad39f5b902e842886c3c818
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492376"
---
## <a name="configure-the-data-source"></a>Configuración del origen de datos

Realice los pasos siguientes para configurar Azure Data Explorer como origen de datos para la herramienta de panel. Trataremos estos pasos con más detalle en esta sección:

1. Cree una entidad de servicio de Azure Active Directory (Azure AD). La herramienta de panel usa la entidad de servicio para acceder al servicio de Azure Data Explorer.

1. Agregue la entidad de servicio de Azure AD al rol *visores* en la base de datos de Azure Data Explorer.

1. Especifique las propiedades de conexión de la herramienta de panel en función de la información de la entidad de servicio de Azure AD y, después, pruebe la conexión.

### <a name="create-a-service-principal"></a>Creación de una entidad de servicio

Puede crear la entidad de servicio en [Azure Portal](#azure-portal) o mediante la experiencia de línea de comandos de la [CLI de Azure](#azure-cli). Independientemente del método que utilice, después de crear la entidad, obtendrá los valores de cuatro propiedades de conexión que usará en pasos posteriores.

#### <a name="azure-portal"></a>Portal de Azure

1. Para crear la entidad de servicio, siga las instrucciones de la [documentación de Azure Portal](/azure/active-directory/develop/howto-create-service-principal-portal).

    1. En la sección [Asignación de la aplicación a un rol](/azure/active-directory/develop/howto-create-service-principal-portal#assign-a-role-to-the-application), asigne un tipo de rol de **lector** al clúster de Azure Data Explorer.

    1. En la sección [Obtención de valores para iniciar sesión](/azure/active-directory/develop/howto-create-service-principal-portal#get-values-for-signing-in), copie los tres valores de propiedad indicados en los pasos: **Id. de directorio** (Id. de inquilino), **Id. de aplicación** y **Contraseña**.

1. En Azure Portal, seleccione **Suscripciones** y copie el identificador de la suscripción en la que creó la entidad de servicio.

    ![Id. de suscripción (Azure Portal)](media/data-explorer-configure-data-source/subscription-id-portal.png)

#### <a name="azure-cli"></a>Azure CLI

1. Crear una entidad de servicio. Establezca un ámbito adecuado y un tipo de rol de `reader`.

    ```azurecli
    az ad sp create-for-rbac --name "https://{UrlToYourDashboard}:{PortNumber}" --role "reader" \
                             --scopes /subscriptions/{SubID}/resourceGroups/{ResourceGroupName}
    ```

    Para más información, vea [Creación de una entidad de servicio de Azure con la CLI de Azure](/cli/azure/create-an-azure-service-principal-azure-cli).

1. El comando devuelve un conjunto de resultados similar al siguiente. Copie los tres valores de propiedad: **id_aplicación**, **contraseña** e **inquilino**.


    ```json
    {
      "appId": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
      "displayName": "{UrlToYourDashboard}:{PortNumber}",
      "name": "https://{UrlToYourDashboard}:{PortNumber}",
      "password": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX",
      "tenant": "XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX"
    }
    ```

1. Obtenga una lista de las suscripciones.

    ```azurecli
    az account list --output table
    ```

    Copie el identificador de suscripción apropiado.

    ![Id. de suscripción (CLI)](media/data-explorer-configure-data-source/subscription-id-cli.png)

### <a name="add-the-service-principal-to-the-viewers-role"></a>Adición de la entidad de servicio al rol visores

Ahora que ya tiene una entidad de servicio, agréguela al rol *visores* en la base de datos de Azure Data Explorer. Puede realizar esta tarea en **Permisos** en Azure Portal o en **Consulta** mediante un comando de administración.

#### <a name="azure-portal---permissions"></a>Azure Portal (Permisos)

1. En Azure Portal, vaya al clúster de Azure Data Explorer.

1. En la sección **Introducción**, seleccione la base de datos con los datos de ejemplo de StormEvents.

    ![Seleccionar base de datos](media/data-explorer-configure-data-source/select-database.png)

1. Seleccione **Permisos** y luego **Agregar**.

    ![Permisos de base de datos](media/data-explorer-configure-data-source/database-permissions.png)

1. En **Add database permissions** (Agregar permisos de base de datos), seleccione el rol **Visor** y después **Select principals** (Seleccionar entidades).

    ![Incorporación de permisos de base de datos](media/data-explorer-configure-data-source/add-permission.png)

1. Busque la entidad de servicio que se ha creado. Seleccione la entidad y después **Seleccionar**.

    ![Administración de permisos en Azure Portal](media/data-explorer-configure-data-source/new-principals.png)

1. Seleccione **Guardar**.

    ![Administración de permisos en Azure Portal](media/data-explorer-configure-data-source/save-permission.png)

#### <a name="management-command---query"></a>Comando de administración (Consulta)

1. En Azure Portal, vaya al clúster de Azure Data Explorer y seleccione **Consulta**.

    ![Consultar](media/data-explorer-configure-data-source/query.png)

1. Ejecute el comando siguiente en la ventana de consulta. Use el identificador de la aplicación y el identificador de inquilino de Azure Portal o la CLI.

    ```kusto
    .add database {TestDatabase} viewers ('aadapp={ApplicationID};{TenantID}')
    ```

    El comando devuelve un conjunto de resultados similar al siguiente. En este ejemplo, la primera fila es para un usuario existente en la base de datos y la segunda fila es para la entidad de servicio que se acaba de agregar.

    ![Conjunto de resultados](media/data-explorer-configure-data-source/result-set.png)
