---
title: Creación de una conexión de datos de Event Hubs en Azure Data Explorer con Python
description: En este artículo, aprenderá a crear una conexión de datos de Event Hubs para Azure Data Explorer con Python.
author: lucygoldbergmicrosoft
ms.author: lugoldbe
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 10/07/2019
ms.openlocfilehash: 36696a6fcdd74dc86705859ba62b608e9aa2ccf9
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492332"
---
# <a name="create-an-event-hub-data-connection-for-azure-data-explorer-by-using-python"></a>Creación de una conexión de datos de Event Hubs en Azure Data Explorer con Python

> [!div class="op_single_selector"]
> * [Portal](ingest-data-event-hub.md)
> * [C#](data-connection-event-hub-csharp.md)
> * [Python](data-connection-event-hub-python.md)
> * [Plantilla de Azure Resource Manager](data-connection-event-hub-resource-manager.md)

En este artículo, va a crear una conexión de datos de Event Hubs para Azure Data Explorer con Python. El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. Azure Data Explorer permite ingerir, o cargar, datos procedentes de Event Hubs, IoT Hubs y blobs escritos en contenedores de blobs.

## <a name="prerequisites"></a>Prerequisites

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

* [Python 3.4+](https://www.python.org/downloads/).

* [Un clúster y una base de datos](create-cluster-database-python.md).

* [Asignación de tablas y columnas](net-standard-ingest-data.md#create-a-table-on-your-test-cluster).

* [Directivas de bases de datos y tablas](database-table-policies-python.md) (opcional).

* [Event Hub con datos para la ingesta](ingest-data-event-hub.md#create-an-event-hub).

[!INCLUDE [data-explorer-data-connection-install-package-python](includes/data-explorer-data-connection-install-package-python.md)]

[!INCLUDE [data-explorer-authentication](includes/data-explorer-authentication.md)]

## <a name="add-an-event-hub-data-connection"></a>Incorporación de una conexión de datos de Event Hubs

En el ejemplo siguiente, se explica cómo se agrega una conexión de datos de Event Hubs mediante programación. Consulte [Conexión a Event Hubs](ingest-data-event-hub.md#connect-to-the-event-hub) para agregar una conexión de datos de Event Hubs en Azure Portal.

```Python
from azure.mgmt.kusto import KustoManagementClient
from azure.mgmt.kusto.models import EventHubDataConnection
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
#The cluster and database that are created as part of the Prerequisites
cluster_name = "mykustocluster"
database_name = "mykustodatabase"
data_connection_name = "myeventhubconnect"
#The event hub that is created as part of the Prerequisites
event_hub_resource_id = "/subscriptions/xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx/resourceGroups/xxxxxx/providers/Microsoft.EventHub/namespaces/xxxxxx/eventhubs/xxxxxx";
consumer_group = "$Default"
location = "Central US"
#The table and column mapping that are created as part of the Prerequisites
table_name = "StormEvents"
mapping_rule_name = "StormEvents_CSV_Mapping"
data_format = "csv"
#Returns an instance of LROPoller, check https://docs.microsoft.com/python/api/msrest/msrest.polling.lropoller?view=azure-python
poller = kusto_management_client.data_connections.create_or_update(resource_group_name=resource_group_name, cluster_name=cluster_name, database_name=database_name, data_connection_name=data_connection_name,
                                        parameters=EventHubDataConnection(event_hub_resource_id=event_hub_resource_id, consumer_group=consumer_group, location=location,
                                                                            table_name=table_name, mapping_rule_name=mapping_rule_name, data_format=data_format))
```

|**Configuración** | **Valor sugerido** | **Descripción del campo**|
|---|---|---|
| tenant_id | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | El identificador de inquilino. También conocido como identificador de directorio.|
| subscriptionId | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | Identificador de suscripción que se usa para la creación de recursos.|
| client_id | *xxxxxxxx-xxxxx-xxxx-xxxx-xxxxxxxxx* | Identificador del cliente de la aplicación que puede acceder a los recursos del inquilino.|
| client_secret | *xxxxxxxxxxxxxx* | Secreto del cliente de la aplicación que puede acceder a los recursos del inquilino. |
| resource_group_name | *testrg* | Nombre del grupo de recursos que contiene el clúster.|
| cluster_name | *mykustocluster* | Nombre del clúster.|
| database_name | *mykustodatabase* | Nombre de la base de datos de destino del clúster.|
| data_connection_name | *myeventhubconnect* | Nombre que desea asignar a la conexión de datos.|
| table_name | *StormEvents* | Nombre de la tabla de destino de la base de datos de destino.|
| mapping_rule_name | *StormEvents_CSV_Mapping* | Nombre de la asignación de columnas asociada a la tabla de destino.|
| data_format | *csv* | Formato de datos del mensaje.|
| event_hub_resource_id | *Identificador del recurso* | Identificador del recurso del centro de eventos que contiene los datos que se van a ingerir. |
| consumer_group | *$Default* | Grupo de consumidores del centro de eventos.|
| ubicación | *Centro de EE. UU.* | Ubicación del recurso de conexión de datos.|

[!INCLUDE [data-explorer-data-connection-clean-resources-python](includes/data-explorer-data-connection-clean-resources-python.md)]
