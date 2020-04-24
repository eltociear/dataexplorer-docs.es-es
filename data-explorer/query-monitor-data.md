---
title: Consulta de datos en Azure Monitor con Azure Data Explorer (versión preliminar)
description: En este tema, se van a consultar los datos en Azure Monitor mediante la creación de un proxy de Azure Data Explorer para consultas entre productos con Application Insights y Log Analytics.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: conceptual
ms.date: 01/28/2020
ms.openlocfilehash: 983a9af42772209df2f48c1b1480e8ff0f34b5d6
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81494196"
---
# <a name="query-data-in-azure-monitor-using-azure-data-explorer-preview"></a>Consulta de datos en Azure Monitor con Azure Data Explorer (versión preliminar)

El clúster de proxy de Azure Data Explorer (ADX Proxy) es una entidad que le permite realizar consultas entre productos entre Azure Data Explorer, [Application Insights (AI)](/azure/azure-monitor/app/app-insights-overview) y [Log Analytics (LA)](/azure/azure-monitor/platform/data-platform-logs) en el servicio [Azure Monitor](/azure/azure-monitor/). Puede asignar áreas de trabajo de Log Analytics o aplicaciones de Application Insights en Azure Monitor como clústeres de proxy. A continuación, puede consultar el clúster de proxy mediante las herramientas de Azure Data Explorer y referirse a él en una consulta entre clústeres. El artículo muestra cómo conectarse a un clúster de proxy, agregar un clúster de proxy a la interfaz de usuario web de Azure Data Explorer y ejecutar consultas en las aplicaciones de Application Insights o en áreas de trabajo de Log Analytics desde Azure Data Explorer.

El flujo del proxy de Azure Data Explorer: 

![Flujo de proxy de ADX](media/adx-proxy/adx-proxy-flow.png)

## <a name="prerequisites"></a>Prerrequisitos

> [!NOTE]
> El proxy de ADX está en modo de versión preliminar. [Conéctese al proxy](#connect-to-the-proxy) para habilitar la característica de proxy de ADX para los clústeres. Póngase en contacto con el equipo de [ADXProxy](mailto:adxproxy@microsoft.com) con cualquier pregunta.

## <a name="connect-to-the-proxy"></a>Conexión con el proxy

1. Compruebe que el clúster nativo de Azure Data Explorer (como el clúster *help*) aparece en el menú izquierdo antes de conectarse al clúster de Log Analytics o de Application Insights.

    ![Clúster nativo de ADX](media/adx-proxy/web-ui-help-cluster.png)

1. En la interfaz de usuario de Azure Data Explorer (https://dataexplorer.azure.com/clusters), seleccione **Agregar clúster**.

1. En la ventana **Agregar clúster**, agregue la dirección URL al clúster de LA o AI. 
    
    * Para LA: `https://ade.loganalytics.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>`
    * Para AI: `https://ade.applicationinsights.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/<ai-app-name>`

    * Seleccione **Agregar**.

    ![Agregar clúster](media/adx-proxy/add-cluster.png)

    Si agrega una conexión a más de un clúster de proxy, dé a cada uno un nombre diferente. En caso contrario, todos tendrán el mismo nombre en el panel izquierdo.

1. Una vez establecida la conexión, el clúster de Log Analytics o Application Insights aparecerá en el panel izquierdo con el clúster de ADX nativo. 

    ![Clústeres de Log Analytics y de Azure Data Explorer](media/adx-proxy/la-adx-clusters.png)

## <a name="run-queries"></a>Ejecutar consultas

Puede ejecutar las consultas usando herramientas de cliente que admitan consultas de Kusto, como: Kusto Explorer, ADX Web UI, Jupyter Kqlmagic, Flow, PowerQuery, PowerShell, Jarvis, Lens, REST API.

> [!TIP]
> * El nombre de la base de datos debe tener el mismo nombre que el recurso especificado en el clúster de proxy. Los nombres distinguen mayúsculas de minúsculas.
> * En consultas entre clústeres, asegúrese de que la nomenclatura de las aplicaciones de Application Insights y las áreas de trabajo de Log Analytics sea correcta.
>     * Si los nombres contienen caracteres especiales, se sustituyen por codificación URL en el nombre del clúster del proxy. 
>     * Si los nombres incluyen caracteres que no cumplen las [reglas del nombre del identificador KQL](kusto/query/schema-entities/entity-names.md), se sustituyen por el carácter de guion **-** .

### <a name="direct-query-from-your-la-or-ai-adx-proxy-cluster"></a>Consulta directa desde el clúster de proxy ADX de LA o AI

Ejecutar consultas en el clúster de LA o AI. Compruebe que el clúster está seleccionado en el panel izquierdo. 

```kusto
Perf | take 10 // Demonstrate query through the proxy on the LA workspace
```

![Consulta del área de trabajo de Log Analytics](media/adx-proxy/query-la.png)

### <a name="cross-query-of-your-la-or-ai-adx-proxy-cluster-and-the-adx-native-cluster"></a>Consulta cruzada del clúster de proxy ADX de LA o AI y el clúster nativo de ADX 

Al ejecutar consultas entre clústeres desde el proxy, compruebe que el clúster nativo de ADX está seleccionado en el panel izquierdo. En los ejemplos siguientes se muestra la combinación de tablas de clúster de ADX (mediante `union`) con el área de trabajo de Log Analytics.

```kusto
union StormEvents, cluster('https://ade.loganalytics.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>').database('<workspace-name>').Perf
| take 10 
```

```kusto
let CL1 = 'https://ade.loganalytics.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>';
union <ADX table>, cluster(CL1).database(<workspace-name>).<table name>
```

   [ ![Consulta cruzada desde el proxy de Azure Data Explorer](media/adx-proxy/cross-query-adx-proxy.png)](media/adx-proxy/cross-query-adx-proxy.png#lightbox)

Usar el [operador `join`](kusto/query/joinoperator.md), en lugar de union, puede requerir una [`hint` (sugerencia)](kusto/query/joinoperator.md#join-hints) para ejecutarla en un clúster nativo de Azure Data Explorer (y no en el proxy). 

## <a name="additional-syntax-examples"></a>Ejemplos de sintaxis adicionales

Las opciones de sintaxis siguientes están disponibles cuando se llama a los clústeres de Application Insights o Log Analytics:

|Descripción de la sintaxis  |Application Insights  |Log Analytics  |
|----------------|---------|---------|
| Base de datos dentro de un clúster que contiene solo el recurso definido en esta suscripción (**recomendado para las consultas entre clústeres**) |   clúster (`https://ade.applicationinsights.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/<ai-app-name>').database('<ai-app-name>`) | clúster (`https://ade.loganalytics.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>').database('<workspace-name>`)     |
| Clúster que contiene todas las aplicaciones o áreas de trabajo en esta suscripción    |     clúster (`https://ade.applicationinsights.io/subscriptions/<subscription-id>`)    |    clúster (`https://ade.loganalytics.io/subscriptions/<subscription-id>`)     |
|Clúster que contiene todas las aplicaciones o áreas de trabajo de la suscripción y que son miembros de este grupo de recursos    |   clúster (`https://ade.applicationinsights.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>`)      |    clúster (`https://ade.loganalytics.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>`)      |
|Clúster que contiene solo el recurso definido en esta suscripción      |    clúster (`https://ade.applicationinsights.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.insights/components/<ai-app-name>`)    |  clúster (`https://ade.loganalytics.io/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/microsoft.operationalinsights/workspaces/<workspace-name>`)     |

## <a name="next-steps"></a>Pasos siguientes

[Escritura de consultas](write-queries.md)
