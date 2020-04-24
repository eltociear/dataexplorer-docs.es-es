---
title: Ingesta de blobs de un extremo a otro en Azure Data Explorer a través de Python
description: En este artículo, aprenderá a ingerir blobs en Azure Data Explorer con un ejemplo de un extremo a otro que usa Python.
author: lucygoldbergmicrosoft
ms.author: lugoldbe
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 02/03/2020
ms.openlocfilehash: 0aa6cd7957dfbd6d76068c8fb4b6c92544e603c1
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492612"
---
# <a name="end-to-end-blob-ingestion-into-azure-data-explorer-through-python"></a>Ingesta de blobs de un extremo a otro en Azure Data Explorer a través de Python

> [!div class="op_single_selector"]
> * [C#](end-to-end-csharp.md)
> * [Python](end-to-end-python.md)
>

Azure Data Explorer es un servicio de exploración de datos escalable y rápido para datos de telemetría y registro. En este artículo se ofrece un ejemplo completo de cómo ingerir datos de Azure Blob Storage en Azure Data Explorer. 

Aprenderá a crear un grupo de recursos mediante programación, una cuenta de almacenamiento y un contenedor, un centro de eventos y un clúster y una base de datos de Azure Data Explorer. También aprenderá a configurar Azure Data Explorer mediante programación para ingerir datos de la nueva cuenta de almacenamiento.

## <a name="prerequisites"></a>Prerrequisitos

Si no tiene una suscripción a Azure, cree una [cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.

## <a name="install-the-python-package"></a>Instalar el paquete de Python

Para instalar el paquete de Python correspondiente a Azure Data Explorer (Kusto), abra un símbolo del sistema que tenga Python en su ruta de acceso. Ejecute estos comandos:

```
pip install azure-common
pip install azure-mgmt-resource
pip install azure-mgmt-kusto
pip install azure-mgmt-eventgrid
pip install azure-kusto-data
pip install azure-storage-blob
```
[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

[!INCLUDE [data-explorer-e2e-event-grid-resource-template](includes/data-explorer-e2e-event-grid-resource-template.md)]

## <a name="code-example"></a>Ejemplo de código 

En el ejemplo de código siguiente se proporciona un proceso paso a paso que da como resultado la ingesta de datos en Azure Data Explorer. 

Primero se crea un grupo de recursos. También se crean los recursos de Azure, como una cuenta de almacenamiento y un contenedor, un centro de eventos y un clúster y una base de datos de Azure Data Explorer, y se agregan entidades de seguridad. A continuación, crea una suscripción de Azure Event Grid, junto con una asignación de tabla y columna, en la base de datos de Azure Data Explorer. Finalmente, crea la conexión de datos para configurar Azure Data Explorer para ingerir datos de la nueva cuenta de almacenamiento.

```python
from azure.common.credentials import ServicePrincipalCredentials
from azure.mgmt.resource import ResourceManagementClient
from azure.mgmt.resource.resources.models import DeploymentMode
import os.path
import json
from azure.kusto.data.request import KustoClient, KustoConnectionStringBuilder
from azure.mgmt.eventgrid import EventGridManagementClient
from azure.mgmt.kusto import KustoManagementClient
from azure.mgmt.kusto.models import EventGridDataConnection

#Directory (tenant) ID
tenant_id = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
#Application ID
client_id = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
#Client secret
client_secret = "xxxxxxxxxxxxxx"
subscription_id = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx"
location = "West Europe"
location_small_case = "westeurope"
#Path to the Azure Resource Manager template JSON from the previous section
azure_resource_template_path = "xxxxxxxxx/template.json";

deployment_name = 'e2eexample'
resource_group_name = deployment_name + "resourcegroup"
event_hub_name = deployment_name + "eventhub"
event_hub_namespace_name = event_hub_name + "ns"
storage_account_name = deployment_name + "storage"
storage_container_name = deployment_name + "storagecontainer"
event_grid_subscription_name = deployment_name + "eventgrid"
kusto_cluster_name = deployment_name + "kustocluster"
kusto_database_name = deployment_name + "kustodatabase"
kusto_table_name = "Events"
kusto_column_mapping_name = "Events_CSV_Mapping"
kusto_data_connection_name = deployment_name + "kustoeventgridconnection"

#principals
principal_id_for_cluster = "xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx";//Application ID
role_for_cluster_principal = "AllDatabasesAdmin";
tenant_id_for_cluster_principal = tenant_id;
principal_type_for_cluster = "App";
principal_id_for_database = "xxxxxxxx@xxxxxxxx.com";//User Email
role_for_database_principal = "Admin";
tenant_id_for_database_principal = tenant_id;
principal_type_for_database = "User";


credentials = ServicePrincipalCredentials(
    client_id=client_id,
    secret=client_secret,
    tenant=tenant_id
)
resource_client = ResourceManagementClient(credentials, subscription_id)

print('Step 1: Create a new resource group in your Azure subscription to manage all the resources for using Azure Data Explorer.')
resource_client.resource_groups.create_or_update(
    resource_group_name,
    {
        'location': location_small_case
    }
)

print('Step 2: Create a Blob Storage, a container in the Storage account, an Event Hub, an Azure Data Explorer cluster, database, and add principals by using an Azure Resource Manager template.')
#Read the Azure Resource Manager template
with open(azure_resource_template_path, 'r') as template_file_fd:
    template = json.load(template_file_fd)

parameters = {
    'eventHubNamespaceName': event_hub_namespace_name,
    'eventHubName': event_hub_name,
    'storageAccountName': storage_account_name,
    'containerName': storage_container_name,
    'kustoClusterName': kusto_cluster_name,
    'kustoDatabaseName': kusto_database_name,
    'principalIdForCluster': principal_id_for_cluster,
    'roleForClusterPrincipal': role_for_cluster_principal,
    'tenantIdForClusterPrincipal': tenant_id_for_cluster_principal,
    'principalTypeForCluster': principal_type_for_cluster,
    'principalIdForDatabase': principal_id_for_database,
    'roleForDatabasePrincipal': role_for_database_principal,
    'tenantIdForDatabasePrincipal': tenant_id_for_database_principal,
    'principalTypeForDatabase': principal_type_for_database
}
parameters = {k: {'value': v} for k, v in parameters.items()}
deployment_properties = {
    'mode': DeploymentMode.incremental,
    'template': template,
    'parameters': parameters
}

#Returns an instance of LROPoller; see https://docs.microsoft.com/python/api/msrest/msrest.polling.lropoller?view=azure-python
poller = resource_client.deployments.create_or_update(
    resource_group_name,
    deployment_name,
    deployment_properties
)
poller.wait()

print('Step 3: Create an Event Grid subscription to publish blob events created in a specific container to an Event Hub.')
event_client = EventGridManagementClient(credentials, subscription_id)
storage_resource_id = '/subscriptions/{}/resourceGroups/{}/providers/Microsoft.Storage/storageAccounts/{}'.format(subscription_id, resource_group_name, storage_account_name)
event_hub_resource_id = '/subscriptions/{}/resourceGroups/{}/providers/Microsoft.EventHub/namespaces/{}/eventhubs/{}'.format(subscription_id, resource_group_name, event_hub_namespace_name, event_hub_name)
event_client.event_subscriptions.create_or_update(storage_resource_id, event_grid_subscription_name, {
    'destination': {
        'endpointType': 'EventHub',
        'properties': {
            'resourceId': event_hub_resource_id
        }
    },
    "filter": {
        "subjectBeginsWith": "/blobServices/default/containers/{}".format(storage_container_name),
        "includedEventTypes": ["Microsoft.Storage.BlobCreated"],
        "advancedFilters": []
    }
})


print('Step 4: Create a table (with three columns: EventTime, EventId, and EventSummary) and column mapping in your Azure Data Explorer database.')
kusto_uri = "https://{}.{}.kusto.windows.net".format(kusto_cluster_name, location_small_case)
database_name = kusto_database_name
kusto_connection_string_builder = KustoConnectionStringBuilder.with_aad_application_key_authentication(connection_string=kusto_uri, aad_app_id=client_id, app_key=client_secret, authority_id=tenant_id)
kusto_client = KustoClient(kusto_connection_string_builder)
create_table_command = ".create table " + kusto_table_name + " (EventTime: datetime, EventId: int, EventSummary: string)"
kusto_client.execute_mgmt(database_name, create_table_command)

create_column_mapping_command = ".create table " + kusto_table_name + " ingestion csv mapping '" + kusto_column_mapping_name \
                                + """' '[{"Name":"EventTime","datatype":"datetime","Ordinal":0},{"Name":"EventId","datatype":"int","Ordinal":1},{"Name":"EventSummary","datatype":"string","Ordinal":2}]'"""
kusto_client.execute_mgmt(database_name, create_column_mapping_command)


print('Step 5: Add an Event Grid data connection. Azure Data Explorer will automatically ingest the data when new blobs are created.')
kusto_management_client = KustoManagementClient(credentials, subscription_id)
data_connections = kusto_management_client.data_connections
#Returns an instance of LROPoller; see https://docs.microsoft.com/python/api/msrest/msrest.polling.lropoller?view=azure-python
poller = data_connections.create_or_update(resource_group_name=resource_group_name, cluster_name=kusto_cluster_name, database_name=kusto_database_name, data_connection_name=kusto_data_connection_name,
                                           parameters=EventGridDataConnection(storage_account_resource_id=storage_resource_id,
                                                                              event_hub_resource_id=event_hub_resource_id, consumer_group="$Default", location=location, table_name=kusto_table_name, mapping_rule_name=kusto_column_mapping_name, data_format="csv"))
poller.wait()
```
|**Configuración** | **Descripción del campo**|
|---|---|---|
| tenant_id | El identificador de inquilino. También se conoce como identificador de directorio.|
| subscription_id | Identificador de suscripción que se usa para la creación de recursos.|
| client_id | Identificador del cliente de la aplicación que puede acceder a los recursos del inquilino.|
| client_secret | Secreto del cliente de la aplicación que puede acceder a los recursos del inquilino. |

## <a name="test-the-code-example"></a>Prueba del ejemplo de código

1. Cargue un archivo en la cuenta de almacenamiento.

    ```python
    account_key = "xxxxxxxxxxxxxx"
    block_blob_service = BlockBlobService(account_name=storage_account_name, account_key=account_key)
    blob_name = "test.csv"
    blob_content = """2007-01-01 00:00:00.0000000,2592,Several trees down
    2007-01-01 00:00:00.0000000,4171,Winter Storm"""
    block_blob_service.create_blob_from_text(container_name=storage_container_name, blob_name=blob_name, text=blob_content)
    ```
    |**Configuración** | **Descripción del campo**|
    |---|---|---|
    | account_key | La clave de acceso de la cuenta de almacenamiento creada mediante programación.|

2. Ejecute una consulta de prueba en Azure Data Explorer.

    ```python
    kusto_uri = "https://{}.{}.kusto.windows.net".format(kusto_cluster_name, location_small_case)
    kusto_connection_string_builder = KustoConnectionStringBuilder.with_aad_application_key_authentication(connection_string=kusto_uri, aad_app_id=client_id, app_key=client_secret, authority_id=tenant_id)
    kusto_client = KustoClient(kusto_connection_string_builder)
    query = "{} | take 10".format(kusto_table_name)
    response = kusto_client.execute_query(kusto_database_name, query)
    print(response.primary_results[0].rows_count)
    ```

## <a name="clean-up-resources"></a>Limpieza de recursos

Utilice el comando siguiente para eliminar el grupo de recursos y limpiar los recursos:

```python
#Returns an instance of LROPoller; see https://docs.microsoft.com/python/api/msrest/msrest.polling.lropoller?view=azure-python
poller = resource_client.resource_groups.delete(resource_group_name=resource_group_name)
poller.wait()
```

## <a name="next-steps"></a>Pasos siguientes

*  Para más información sobre otras formas de crear un clúster y una base de datos, consulte [Creación de un clúster y una base de datos de Azure Data Explorer mediante C#](create-cluster-database-python.md).
* Para más información sobre los métodos de ingesta, consulte [Ingesta de datos de Azure Data Explorer](ingest-data-overview.md).
* Para información acerca de la aplicación web, consulte [Inicio rápido: Consulta de datos en la interfaz de usuario web de Azure Data Explorer](web-query-data.md).
* [Escriba consultas](write-queries.md) con el lenguaje de consulta Kusto.