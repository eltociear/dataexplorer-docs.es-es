---
title: Conexión a Azure Data Explorer desde Azure Databricks
description: En este tema se muestra cómo usar Azure Databricks para acceder a los datos de Azure Data Explorer.
author: manojraheja
ms.author: maraheja
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 05/21/2020
ms.openlocfilehash: 11db44424b86e4ca946ea104301bcd074797ca40
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83863207"
---
# <a name="connect-to-azure-data-explorer-from-azure-databricks"></a>Conexión a Azure Data Explorer desde Azure Databricks

[Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/what-is-azure-databricks) es una plataforma de análisis basada en Apache Spark que está optimizada para la plataforma de Microsoft Azure. En este artículo se muestra cómo usar Azure Databricks para acceder a los datos de Azure Data Explorer. Hay varias maneras de realizar la autenticación con Azure Data Explorer, incluidos un inicio de sesión del dispositivo y una aplicación de Azure Active Directory (Azure AD).
 
## <a name="prerequisites"></a>Requisitos previos

- [Creación de un clúster y de la base de datos de Azure Data Explorer](create-cluster-database-portal.md).
- [Creación de un área de trabajo de Azure Databricks](/azure/azure-databricks/quickstart-create-databricks-workspace-portal#create-an-azure-databricks-workspace). En **Servicio de Azure Databricks**, en la lista desplegable **Plan de tarifa**, seleccione **Premium**. Esta selección le permite usar secretos de Azure Databricks para almacenar sus credenciales y hacer referencia a ellas en cuadernos y trabajos.

- [Creación de un clúster](https://docs.azuredatabricks.net/user-guide/clusters/create.html) en Azure Databricks con la configuración predeterminada.

 ## <a name="install-the-kusto-spark-connector-on-your-azure-databricks-cluster"></a>Instalación del conector de Spark de Kusto en un clúster de Azure Databricks

Para instalar [spark-kusto-connector](https://mvnrepository.com/artifact/com.microsoft.azure.kusto/spark-kusto-connector) en un clúster de Azure Databricks:

1. Vaya al área de trabajo de Azure Databricks y [cree una biblioteca](https://docs.azuredatabricks.net/user-guide/libraries.html#create-a-library).
1. Busque el paquete *spark-kusto-connector* en Maven Central, instale la versión más reciente y conéctese al clúster. 

## <a name="connect-to-azure-data-explorer-by-using-a-device-authentication"></a>Conexión a Azure Data Explorer mediante una autenticación de dispositivo

[Código de ejemplo](https://github.com/Azure/azure-kusto-spark/blob/master/samples/src/main/python/pyKusto.py).

## <a name="connect-to-azure-data-explorer-by-using-an-azure-ad-app"></a>Conexión a Azure Data Explorer mediante una aplicación de Azure AD

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

### <a name="store-and-secure-your-azure-ad-app-id-and-key-optional"></a>Almacenamiento y protección de la clave y el identificador de aplicación de Azure AD (opcional)  

Almacene y proteja la clave y el identificador de aplicación de Azure AD mediante [secretos](https://docs.azuredatabricks.net/user-guide/secrets/index.html#secrets) de Azure Databricks como se indica a continuación:

1. [Configure la CLI](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#set-up-the-cli).
1. [Instale la CLI](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#install-the-cli). 
1. [Configure la autenticación](https://docs.azuredatabricks.net/user-guide/dev-tools/databricks-cli.html#set-up-authentication).
1. Configure los [secretos](https://docs.azuredatabricks.net/user-guide/secrets/index.html#secrets) mediante los siguientes comandos de ejemplo:

    ```databricks secrets create-scope --scope adx```

    ```databricks secrets put --scope adx --key myaadappid```

    ```databricks secrets put --scope adx --key myaadappkey```

    ```databricks secrets list --scope adx```

### <a name="sample-code"></a>Código de ejemplo

1. [Código de ejemplo](https://github.com/Azure/azure-kusto-spark/blob/master/samples/src/main/python/pyKusto.py). 
1. Actualice los valores de marcador de posición con el nombre del clúster, el nombre de la base de datos, el nombre de la tabla, el identificador del inquilino de Azure AD y la clave de la aplicación de AAD. Si guarda sus credenciales en el almacén de secretos de Databricks, actualice el código según corresponda para recuperar los valores de dbutils.
