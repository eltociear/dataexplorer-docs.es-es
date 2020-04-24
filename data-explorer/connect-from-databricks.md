---
title: Conexión a Azure Data Explorer desde Azure Databricks mediante Python
description: En este tema aprenderá a utilizar la biblioteca de Python en Azure Databricks para obtener acceso a los datos de Azure Data Explorer mediante uno de los dos métodos de autenticación.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 11/27/2018
ms.openlocfilehash: 7e1c7dd313f42884132fe014367c0402418be708
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492908"
---
# <a name="connect-to-azure-data-explorer-from-azure-databricks-by-using-python"></a>Conexión a Azure Data Explorer desde Azure Databricks mediante Python

[Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/what-is-azure-databricks) es una plataforma de análisis basada en Apache Spark que está optimizada para la plataforma de Microsoft Azure. En este artículo se muestra cómo usar una biblioteca de Python en Azure Databricks para obtener acceso a los datos de Azure Data Explorer. Hay varias maneras de realizar la autenticación con Azure Data Explorer, incluidos un inicio de sesión del dispositivo y una aplicación de Azure Active Directory (Azure AD).

## <a name="prerequisites"></a>Prerequisites

- [Creación de un clúster y de la base de datos de Azure Data Explorer](/azure/data-explorer/create-cluster-database-portal).
- [Creación de un área de trabajo de Azure Databricks](/azure/azure-databricks/quickstart-create-databricks-workspace-portal#create-an-azure-databricks-workspace). En **Servicio de Azure Databricks**, en la lista desplegable **Plan de tarifa**, seleccione **Premium**. Esta selección le permite usar secretos de Azure Databricks para almacenar sus credenciales y hacer referencia a ellas en cuadernos y trabajos.

- [Creación de un clúster](https://docs.azuredatabricks.net/user-guide/clusters/create.html) en Azure Databricks con las especificaciones siguientes (configuración mínima necesaria para ejecutar los cuadernos de ejemplo):

   ![Especificaciones para la creación de un clúster](media/connect-from-databricks/databricks-create-cluster.png)

## <a name="install-the-python-library-on-your-azure-databricks-cluster"></a>Instalación de la biblioteca de Python en el clúster de Azure Databricks

Para instalar la [biblioteca de Python](kusto/api/python/kusto-python-client-library.md) en el clúster de Azure Databricks:

1. Vaya al área de trabajo de Azure Databricks y [cree una biblioteca](https://docs.azuredatabricks.net/user-guide/libraries.html#create-a-library).
2. [Cargue un paquete PyPI de Python o Python Egg](https://docs.azuredatabricks.net/user-guide/libraries.html#upload-a-python-pypi-package-or-python-egg).
   - Cargue, instale y adjunte la biblioteca en el clúster de Databricks.
   - Escriba el nombre de PyPi: **azure-kusto-data**.

## <a name="connect-to-azure-data-explorer-by-using-a-device-login"></a>Conexión a Azure Data Explorer mediante un inicio de sesión del dispositivo

[Importe un cuaderno](https://docs.azuredatabricks.net/user-guide/notebooks/notebook-manage.html#import-a-notebook) mediante el cuaderno [Query-ADX-device-login](https://github.com/Azure/azure-kusto-docs-samples/blob/master/Databricks_notebooks/Query-ADX-device-login.ipynb). A continuación, puede conectarse a Azure Data Explorer con sus credenciales.

## <a name="connect-to-adx-by-using-an-azure-ad-app"></a>Conexión a ADX mediante una aplicación de Azure AD

1. Cree aplicaciones de Azure AD mediante el [aprovisionamiento de una aplicación de Azure AD](kusto/management/access-control/how-to-provision-aad-app.md).
1. Conceda acceso a la aplicación de Azure AD en la base de datos de Azure Data Explorer como se indica a continuación:

    ```kusto
    .set database <DB Name> users ('aadapp=<AAD App ID>;<AAD Tenant ID>') 'AAD App to connect Spark to ADX
    ```
    |   |   |
    | - | - |
    | ```DB Name``` | Nombre de la base de datos |
    | ```AAD App ID``` | Identificador de aplicación de Azure AD |
    | ```AAD Tenant ID``` | Identificador de inquilino de Azure AD |

### <a name="find-your-azure-ad-tenant-id"></a>Búsqueda del identificador de inquilino de Azure AD

Para autenticar una aplicación, Azure Data Explorer usa el identificador de inquilino de Azure AD. Para buscar el identificador de inquilino, use la siguiente dirección URL. Sustituya *YourDomain* por su dominio.

```
https://login.windows.net/<YourDomain>/.well-known/openid-configuration/
```

Por ejemplo, si el nombre de dominio es *contoso.com*, la dirección URL es: [https://login.windows.net/contoso.com/.well-known/openid-configuration/](https://login.windows.net/contoso.com/.well-known/openid-configuration/). Seleccione esta dirección URL para ver los resultados. La primera línea se muestra del modo siguiente: 

```
"authorization_endpoint":"https://login.windows.net/6babcaad-604b-40ac-a9d7-9fd97c0b779f/oauth2/authorize"
```

El identificador de inquilino es `6babcaad-604b-40ac-a9d7-9fd97c0b779f`. 

### <a name="store-and-secure-your-azure-ad-app-id-and-key"></a>Almacenamiento y protección de la clave y el identificador de aplicación de Azure AD 

Almacene y proteja la clave y el identificador de aplicación de Azure AD mediante [secretos](https://docs.azuredatabricks.net/user-guide/secrets/index.html#secrets) de Azure Databricks como se indica a continuación:
1. [Configure la CLI](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#set-up-the-cli).
1. [Instale la CLI](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#install-the-cli). 
1. [Configure la autenticación](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#set-up-authentication).
1. Configure los [secretos](https://docs.azuredatabricks.net/user-guide/secrets/index.html#secrets) mediante los siguientes comandos de ejemplo:

    ```databricks secrets create-scope --scope adx```

    ```databricks secrets put --scope adx --key myaadappid```

    ```databricks secrets put --scope adx --key myaadappkey```

    ```databricks secrets list --scope adx```

### <a name="import-a-notebook"></a>Importación de un cuaderno
[Importe un cuaderno](https://docs.azuredatabricks.net/user-guide/notebooks/notebook-manage.html#import-a-notebook) mediante el cuaderno [Query-ADX-AAD-App](https://github.com/Azure/azure-kusto-docs-samples/blob/master/Databricks_notebooks/Query-ADX-AAD-App.ipynb) para conectarse a Azure Data Explorer. Actualice los valores de marcador de posición con el nombre del clúster, el nombre de la base de datos y el identificador de inquilino de Azure AD.
