---
title: Creación de un clúster y una base de datos de Azure Data Explorer con la CLI de Azure
description: Aprenda a crear un clúster y una base de datos de Azure Data Explorer mediante la CLI de Azure.
author: radennis
ms.author: radennis
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 06/03/2019
ms.openlocfilehash: b7e8611ba6427880f15d57137e31010047c39e01
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83224619"
---
# <a name="create-an-azure-data-explorer-cluster-and-database-by-using-azure-cli"></a>Creación de un clúster y una base de datos de Azure Data Explorer mediante la CLI de Azure

> [!div class="op_single_selector"]
> * [Portal](create-cluster-database-portal.md)
> * [CLI](create-cluster-database-cli.md)
> * [PowerShell](create-cluster-database-powershell.md)
> * [C#](create-cluster-database-csharp.md)
> * [Python](create-cluster-database-python.md)
> * [Plantilla ARM](create-cluster-database-resource-manager.md)

Azure Data Explorer es un servicio de análisis de datos rápido y totalmente administrado para analizar en tiempo real grandes volúmenes de datos de que se transmiten desde aplicaciones, sitios web, dispositivos IoT, etc. Para usar Azure Data Explorer, cree primero un clúster y una o varias bases de datos en ese clúster. A continuación, ingerirá (cargará) los datos en una base de datos para que pueda ejecutar consultas en ella. En este artículo, se crean un clúster y una base de datos mediante la CLI de Azure.

## <a name="prerequisites"></a>Prerrequisitos

Para completar este artículo, se necesita una suscripción de Azure. Si no tiene una, [cree una cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

[!INCLUDE [cloud-shell-try-it.md](includes/cloud-shell-try-it.md)]

Si decide instalar y usar la CLI de Azure localmente, para este artículo es preciso la CLI de Azure versión 2.0.4 o posterior. Ejecute `az --version` para comprobar la versión. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

## <a name="configure-the-cli-parameters"></a>Configuración de los parámetros de la CLI

Los pasos siguientes no son necesarios si ejecuta comandos en Azure Cloud Shell. Si ejecuta la CLI localmente, realice los pasos siguientes para iniciar sesión en Azure y establecer su suscripción actual:

1. Ejecute el siguiente comandos para iniciar sesión en Azure:

    ```azurecli-interactive
    az login
    ```

1. Establezca la suscripción donde quiere que se cree el clúster. Reemplace `MyAzureSub` por el nombre de la suscripción de Azure que quiere usar:

    ```azurecli-interactive
    az account set --subscription MyAzureSub
    ```
   
1. Instale la extensión para usar la versión más reciente de la CLI de Kusto:

    ```azurecli-interactive
    az extension add -n kusto
    ```

## <a name="create-the-azure-data-explorer-cluster"></a>Creación del clúster de Azure Data Explorer

1. Cree el clúster mediante el siguiente comando:

    ```azurecli-interactive
    az kusto cluster create --cluster-name azureclitest --sku name="Standard_D13_v2" tier="Standard" --resource-group testrg --location westus
    ```

   |**Configuración** | **Valor sugerido** | **Descripción del campo**|
   |---|---|---|
   | name | *azureclitest* | Nombre que quiere para el clúster.|
   | sku | *Standard_D13_v2* | La SKU que se usará para el clúster. Parámetros: *name*: el nombre de SKU. *tier*: el nivel de SKU. |
   | resource-group | *testrg* | Nombre del grupo de recursos en el que se creará el clúster. |
   | ubicación | *westus* | La ubicación en la que se creará el clúster. |

    Hay varios parámetros opcionales que puede usar, como la capacidad del clúster, etcétera.

1. Ejecute el siguiente comando para comprobar si el clúster se creó correctamente:

    ```azurecli-interactive
    az kusto cluster show --cluster-name azureclitest --resource-group testrg
    ```

Si el resultado contiene `provisioningState` con el valor `Succeeded`, significa que el clúster se ha creado correctamente.

## <a name="create-the-database-in-the-azure-data-explorer-cluster"></a>Creación de la base de datos en el clúster de Azure Data Explorer

1. Cree la base de datos con el siguiente comando:

    ```azurecli-interactive
    az kusto database create --cluster-name azureclitest --database-name clidatabase --resource-group testrg --read-write-database soft-delete-period=P365D hot-cache-period=P31D location=westus
    ```

   |**Configuración** | **Valor sugerido** | **Descripción del campo**|
   |---|---|---|
   | cluster-name | *azureclitest* | Nombre del clúster donde se creará la base de datos.|
   | database-name | *clidatabase* | Nombre de la base de datos.|
   | resource-group | *testrg* | Nombre del grupo de recursos en el que se creará el clúster. |
   | read-write-database | *P365D* *P31D* *westus* | El tipo de base de datos. Parámetros: *soft-delete-period*, indica la cantidad de tiempo que los datos estarán disponibles para consulta. Para más información, consulte [Directiva de retención](kusto/management/retentionpolicy.md). *hot-cache-period*: cantidad de tiempo que los datos se conservarán en la caché. Para más información, consulte [Directiva de caché](kusto/management/cachepolicy.md). *location*: esta es la ubicación en la que se creará la base de datos. |

1. Ejecute el siguiente comando para ver la base de datos que ha creado:

    ```azurecli-interactive
    az kusto database show --database-name clidatabase --resource-group testrg --cluster-name azureclitest
    ```

Ahora cuenta con un clúster y una base de datos.

## <a name="clean-up-resources"></a>Limpieza de recursos

* Si tiene previsto seguir nuestros otros artículos, conserve los recursos que creó.
* Para limpiar los recursos, elimine el clúster. Cuando se elimina un clúster, también se eliminan todas las bases de datos en él. Use el siguiente comando para eliminar el clúster:

    ```azurecli-interactive
    az kusto cluster delete --cluster-name azureclitest --resource-group testrg
    ```

## <a name="next-steps"></a>Pasos siguientes

* [Ingesta de datos mediante la biblioteca de Python de Azure Data Explorer](python-ingest-data.md)
