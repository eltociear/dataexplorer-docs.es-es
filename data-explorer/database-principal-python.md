---
title: Incorporación de entidades de seguridad de base de datos para Azure Data Explorer con Python
description: En este artículo, aprenderá a agregar entidades de seguridad de base de datos para Azure Data Explorer con Python.
author: lucygoldbergmicrosoft
ms.author: lugoldbe
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 02/03/2020
ms.openlocfilehash: 3ddb34692158fdc7e6d4f7786f84b8c059bc0621
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493368"
---
# <a name="add-database-principals-for-azure-data-explorer-by-using-python"></a>Incorporación de entidades de seguridad de base de datos para Azure Data Explorer con Python

> [!div class="op_single_selector"]
> * [C#](database-principal-csharp.md)
> * [Python](database-principal-python.md)
> * [Plantilla de Azure Resource Manager](database-principal-resource-manager.md)

El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. En este artículo, agregará entidades de seguridad de base de datos para Azure Data Explorer con Python.

## <a name="prerequisites"></a>Prerequisites

* Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* [Cree un clúster y una base de datos](create-cluster-database-python.md)

## <a name="install-python-package"></a>Instalación del paquete de Python

Para instalar el paquete de Python correspondiente a Azure Data Explorer (Kusto), abra un símbolo del sistema que tenga Python en su ruta de acceso. Ejecute este comando:

```
pip install azure-common
pip install azure-mgmt-kusto
```

[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

## <a name="add-a-database-principal"></a>Incorporación de una entidad de seguridad de base de datos

En el ejemplo siguiente, se explica cómo se agrega una entidad de seguridad de base de datos mediante programación.

```Python
from azure.mgmt.kusto import KustoManagementClient
from azure.mgmt.kusto.models import DatabasePrincipalAssignment
from azure.common.credentials import ServicePrincipalCredentials

#Directory (tenant) ID
tenant_id = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
#Application ID
client_id = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
#Client Secret
client_secret = "xxxxxxxxxxxxxx"
subscription_id = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
credentials = ServicePrincipalCredentials(
        client_id=client_id,
        secret=client_secret,
        tenant=tenant_id
    )
kusto_management_client = KustoManagementClient(credentials, subscription_id)

resource_group_name = "testrg"
#The cluster and database that is created as part of the Prerequisites
cluster_name = "mykustocluster"
database_name = "mykustodatabase"
principal_assignment_name = "clusterPrincipalAssignment1"
#User email, application ID, or security group name
principal_id = "xxxxxxxx"
#AllDatabasesAdmin or AllDatabasesViewer
role = "AllDatabasesAdmin"
tenant_id_for_principal = tenantId
#User, App, or Group
principal_type = "App"

#Returns an instance of LROPoller, check https://docs.microsoft.com/python/api/msrest/msrest.polling.lropoller?view=azure-python
poller = kusto_management_client.database_principal_assignments.create_or_update(resource_group_name=resource_group_name, cluster_name=cluster_name, database_name=database_name, principal_assignment_name= principal_assignment_name, parameters=DatabasePrincipalAssignment(principal_id=principal_id, role=role, tenant_id=tenant_id_for_principal, principal_type=principal_type))
```

|**Configuración** | **Valor sugerido** | **Descripción del campo**|
|---|---|---|
| tenant_id | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | El identificador de inquilino. También conocido como identificador de directorio.|
| subscription_id | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | Identificador de suscripción que se usa para la creación de recursos.|
| client_id | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | Identificador del cliente de la aplicación que puede acceder a los recursos del inquilino.|
| client_secret | *xxxxxxxxxxxxxx* | Secreto del cliente de la aplicación que puede acceder a los recursos del inquilino. |
| resource_group_name | *testrg* | Nombre del grupo de recursos que contiene el clúster.|
| cluster_name | *mykustocluster* | Nombre del clúster.|
| database_name | *mykustodatabase* | Nombre de la base de datos.|
| principal_assignment_name | *databasePrincipalAssignment1* | Nombre del recurso principal de base de datos.|
| principal_id | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | El identificador de la entidad de seguridad, que puede ser el correo electrónico del usuario, el identificador de la aplicación o el nombre del grupo de seguridad.|
| rol | *Administrador* | El rol de la entidad de seguridad de base de datos, que puede ser "Admin" (Administrador), "Ingestor" (Agente de ingesta), "Monitor" (Monitor), "User" (Usuario), "UnrestrictedViewers" (Visores no restringidos) o "Viewer" (Visor).|
| tenant_id_for_principal | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | El identificador de inquilino de la entidad de seguridad.|
| principal_type | *Aplicación* | El tipo de la entidad de seguridad, que puede ser "User" (Usuario), "App" (Aplicación) o "Group" (Grupo)|

## <a name="next-steps"></a>Pasos siguientes

* [Ingesta de datos mediante la biblioteca de Python de Azure Data Explorer](python-ingest-data.md)
