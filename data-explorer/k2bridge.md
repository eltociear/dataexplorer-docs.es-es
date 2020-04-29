---
title: Visualización de datos de Azure Data Explorer mediante Kibana
description: En este artículo, aprenderá a configurar Azure Data Explorer como origen de datos de Kibana.
author: orspod
ms.author: orspodek
ms.reviewer: guregini
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/12/2020
ms.openlocfilehash: 7389bfdf437d5fc6e4872f9f35ed40d5cb7b2f16
ms.sourcegitcommit: e1e35431374f2e8b515bbe2a50cd916462741f49
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/23/2020
ms.locfileid: "82108378"
---
# <a name="visualize-data-from-azure-data-explorer-in-kibana-with-the-k2bridge-open-source-connector"></a>Visualización de datos de Azure Data Explorer en Kibana con el conector de código abierto K2Bridge

K2Bridge (Kibana-Kusto Bridge) permite usar Azure Data Explorer como origen de datos y visualizar dichos datos en Kibana. K2Bridge es una aplicación en contenedores de [código abierto](https://github.com/microsoft/K2Bridge). Actúa como un proxy entre una instancia de Kibana y un clúster de Azure Data Explorer. En este artículo se describe cómo usar K2Bridge para crear esa conexión.

K2Bridge traduce las consultas de Kibana al lenguaje de consulta de Kusto (KQL) y devuelve a Kibana los resultados de Azure Data Explorer.

   ![Conexión Kibana con Azure Data Explorer mediante K2Bridge](media/k2bridge/k2bridge-chart.png)

K2Bridge admite la pestaña **Discover** (Detectar) de Kibana, donde puede hacer lo siguiente:

* Buscar y explorar los datos.
* Filtrar los resultados.
* Agregar o quitar campos en la cuadrícula de resultados.
* Ver el contenido del registro.
* Guardar y compartir búsquedas.

En la siguiente imagen se muestra una instancia de Kibana enlazada a Azure Data Explorer mediante K2Bridge. La experiencia de usuario en Kibana no cambia.

   [![Página de Kibana enlazada a Azure Data Explorer](media/k2bridge/k2bridge-kibana-page.png)](media/k2bridge/k2bridge-kibana-page.png#lightbox)

## <a name="prerequisites"></a>Prerrequisitos

Para poder visualizar datos de Azure Data Explorer en Kibana, debe tener a punto lo siguiente:

* [Helm v3](https://github.com/helm/helm#install), que es el administrador de paquetes de Kubernetes.

* Clúster de Azure Kubernetes Service (AKS) o cualquier otro clúster de Kubernetes. Las versiones 1.14 a 1.16 se han probado y comprobado. Si necesita un clúster de AKS, consulte los artículos en los que se explica cómo implementar un clúster de AKS [mediante la CLI de Azure](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough) o [mediante Azure Portal](https://docs.microsoft.com/azure/aks/kubernetes-walkthrough-portal).

* Un [clúster de Azure Data Explorer](create-cluster-database-portal.md), que incluye la dirección URL del clúster y el nombre de la base de datos.

* Una entidad de servicio de Azure Active Directory (Azure AD) autorizada para ver datos en Azure Data Explorer, como por ejemplo, el identificador de cliente y el secreto de cliente.

    Se recomienda usar una entidad de servicio con permiso de visor, no es preciso que tenga permisos de nivel superior. [Establezca los permisos de visualización del clúster para la entidad de servicio de Azure AD](https://docs.microsoft.com/azure/data-explorer/manage-database-permissions#manage-permissions-in-the-azure-portal).

    Para más información, sobre la entidad de servicio de Azure AD, consulte [Creación de una entidad de servicio de Azure AD](https://docs.microsoft.com/azure/active-directory/develop/howto-create-service-principal-portal#create-an-azure-active-directory-application).

## <a name="run-k2bridge-on-azure-kubernetes-service-aks"></a>Ejecución de K2Bridge en Azure Kubernetes Service (AKS)

De forma predeterminada, el gráfico de Helm de K2Bridge hace referencia a una imagen disponible públicamente ubicada en el Registro de contenedor de Microsoft (MCR). MCR no requiere credenciales.

1. Descargue los gráficos de Helm necesarios.

1. Agregue la dependencia Elasticsearch a Helm. La dependencia es necesaria porque K2Bridge utiliza una pequeña instancia interna de Elasticsearch. La instancia atiende solicitudes relacionadas con los metadatos, como las consultas de patrones de índice y las consultas guardadas. Esta instancia interna no guarda datos empresariales y se puede considerar un detalle de implementación.

    1. Para agregar la dependencia de Elasticsearch a Helm, ejecute estos comandos:

        ```bash
        helm repo add elastic https://helm.elastic.co
        helm repo update
        ```

    1. Para obtener el gráfico de K2Bridge del directorio de gráficos del repositorio de GitHub realice estas acciones:

        1. Clonar el repositorio de [GitHub](https://github.com/microsoft/K2Bridge).
        1. Ir al directorio raíz del repositorio de K2Bridge.
        1. Ejecute este comando:

            ```bash
            helm dependency update charts/k2bridge
            ```

1. Implemente K2Bridge.

    1. Establezca las variables en los valores correctos para su entorno.

        ```bash
        ADX_URL=[YOUR_ADX_CLUSTER_URL] #For example, https://mycluster.westeurope.kusto.windows.net
        ADX_DATABASE=[YOUR_ADX_DATABASE_NAME]
        ADX_CLIENT_ID=[SERVICE_PRINCIPAL_CLIENT_ID]
        ADX_CLIENT_SECRET=[SERVICE_PRINCIPAL_CLIENT_SECRET]
        ADX_TENANT_ID=[SERVICE_PRINCIPAL_TENANT_ID]
        ```

    1. De forma opción, habilite la telemetría de Application Insights. Si es la primera vez que usa Azure Application Insights, [cree un recurso de Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/create-new-resource). [Copie la clave de instrumentación](https://docs.microsoft.com/azure/azure-monitor/app/create-new-resource#copy-the-instrumentation-key) en una variable.

        ```bash
        APPLICATION_INSIGHTS_KEY=[INSTRUMENTATION_KEY]
        COLLECT_TELEMETRY=true
        ```

    1. <a name="install-k2bridge-chart"></a>Instale el gráfico de K2Bridge.

        ```bash
        helm install k2bridge charts/k2bridge -n k2bridge --set image.repository=$REPOSITORY_NAME/$CONTAINER_NAME --set settings.adxClusterUrl="$ADX_URL" --set settings.adxDefaultDatabaseName="$ADX_DATABASE" --set settings.aadClientId="$ADX_CLIENT_ID" --set settings.aadClientSecret="$ADX_CLIENT_SECRET" --set settings.aadTenantId="$ADX_TENANT_ID" [--set image.tag=latest] [--set privateRegistry="$IMAGE_PULL_SECRET_NAME"] [--set settings.collectTelemetry=$COLLECT_TELEMETRY]
        ```

        En [Configuration](https://github.com/microsoft/K2Bridge/blob/master/docs/configuration.md) (Configuración) puede encontrar el conjunto completo de opciones de configuración.

    1. La salida del comando anterior sugiere el siguiente comando de Helm para implementar Kibana. Opcionalmente, ejecute este comando:

        ```bash
        helm install kibana elastic/kibana -n k2bridge --set image=docker.elastic.co/kibana/kibana-oss --set imageTag=6.8.5 --set elasticsearchHosts=http://k2bridge:8080
        ```

    1. Use el reenvío de puertos para acceder a Kibana en localhost.

        ```bash
        kubectl port-forward service/kibana-kibana 5601 --namespace k2bridge
        ```

    1. Para conectarse a Kibana, use la siguiente URL http://127.0.0.1:5601.

    1. Exponga Kibana a los usuarios. Dispone de varios métodos para hacerlo. El mejor método dependerá en gran medida del caso de uso.

        Por ejemplo, puede exponer el servicio como un servicio de Load Balancer. Para ello, agregue el parámetro **--set service.type=LoadBalancer** al [comando **install** de Helm de K2Bridge anterior](#install-k2bridge-chart).

        Luego, ejecute este comando:

        ```bash
        kubectl get service -w -n k2bridge
        ```

        El resultado debe ser similar al siguiente:

        ```bash
        NAME            TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
        kibana-kibana   LoadBalancer   xx.xx.xx.xx    <pending>     5601:30128/TCP   4m24s
        ```

        Después, puede utilizar el valor de EXTERNAL-IP generado que aparece. Úselo para acceder a Kibana abriendo un explorador y yendo a \<EXTERNAL-IP\>:5601.

1. Configure patrones de índice para acceder a sus datos.

    En una nueva instancia de Kibana:

    1. Abra Kibana.
    1. Vaya a **Management** (Administración).
    1. Seleccione **Index Patterns** (Patrones de índice).
    1. Cree un patrón de índice. El nombre del índice debe coincidir exactamente con de la tabla o la función, sin asterisco (\*). Puede copiar la línea correspondiente de la lista.

> [!Note]
> Para ejecutar K2Bridge en otros proveedores de Kubernetes, cambie el valor **storageClassName** de Elasticsearch en values.yaml para que se ajuste al que sugiere el proveedor.

## <a name="visualize-data"></a>Visualización de datos

Si Azure Data Explorer está configurado como un origen de datos para Kibana, puede usar Kibana para explorar los datos.

1. En Kibana, en el menú de la izquierda, seleccione la pestaña **Discover** (Descubrir).

1. En el cuadro de lista desplegable situado más a la izquierda, seleccione un patrón de índice. El patrón define el origen de datos que se desea explorar. En este caso, el patrón de índice es una tabla de Azure Data Explorer.

   ![Selección de patrón de índice](media/k2bridge/k2bridge-select-an-index-pattern.png)

1. Si sus datos tienen un campo de filtro de tiempo, puede especificar el intervalo de tiempo. En la parte superior derecha de la página **Discover** (Detectar), seleccione un filtro de tiempo. De forma predeterminada, la página muestra los datos de los últimos 15 minutos.

   ![Selección de un filtro de tiempo](media/k2bridge/k2bridge-time-filter.png)

1. La tabla de resultados muestra los primeros 500 registros. Puede expandir un documento para examinar los datos de sus campos en formato JSON o de tabla.

   ![Un registro expandido](media/k2bridge/k2bridge-expand-record.png)

1. De manera predeterminada, la tabla de resultados incluye la columna **_source**. También incluye la columna **Time**, en caso de que exista el campo de tiempo. Para elegir columnas específicas para agregarlas a la tabla de resultados, seleccione **add** (Agregar) junto al nombre del campo en el panel de la izquierda.

   ![Columnas concretas con el botón para agregar resaltado](media/k2bridge/k2bridge-specific-columns.png)

1. En la barra de consulta, puede realizar las acciones siguientes para buscar los datos:

    * Escribir un término de búsqueda.
    * Usar la sintaxis de consulta de Lucene. Por ejemplo:
        * Busque "error" para encontrar todos los registros que contienen este valor.
        * Busque "estado: 200" para obtener todos los registros con un valor de estado de 200.
    * Usar los operadores lógicos **AND**, **OR** y **NOT**.
    * Usar los caracteres comodín, asterisco (\*) y signo de interrogación (?). Por ejemplo, la consulta "destination_city: L*" coincide con los registros en que el valor de destination-city comienza por "L" o "l" (K2Bridge no distingue mayúsculas de minúsculas).

    ![Ejecución de una consulta](media/k2bridge/k2bridge-run-query.png)

    > [!Tip]
    > En [Búsqueda](https://github.com/microsoft/K2Bridge/blob/master/docs/searching.md), puede encontrar más reglas de búsqueda y lógica.

1. Para filtrar los resultados de la búsqueda, use la lista de campos del panel de la derecha de la página. La lista de campos es donde puede ver:

    * Los primeros cinco valores del campo.
    * El número de registros que contienen el campo.
    * El porcentaje de registros que contienen cada valor.

    >[!Tip]
    > Use el icono de la lupa para buscar todos los registros que tienen un valor específico.

    ![Lista de campos con la lupa resaltada](media/k2bridge/k2bridge-field-list.png)

    La lupa también se puede usar para filtrar los resultados y usar la vista en formato de tabla de cada registro en la tabla de resultados.

     ![Lista de tablas con la lupa resaltada](media/k2bridge/k2bridge-table-list.png)

1. Seleccione las opciones **Save** (Guardar) o **Share** (Compartir) para la búsqueda.

     ![Botones resaltados para compartir la búsqueda](media/k2bridge/k2bridge-save-search.png)
