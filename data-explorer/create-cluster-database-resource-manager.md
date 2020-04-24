---
title: Creación de un clúster y una base de datos de Azure Data Explorer mediante una plantilla de Azure Resource Manager
description: Aprenda a crear un clúster y una base de datos de Azure Data Explorer mediante una plantilla de Azure Resource Manager
author: orspod
ms.author: orspodek
ms.reviewer: lugoldbe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 09/26/2019
ms.openlocfilehash: 7a12f43c5c6685ea474f6f19f6e585b09b194fce
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81492692"
---
# <a name="create-an-azure-data-explorer-cluster-and-database-by-using-an-azure-resource-manager-template"></a>Creación de un clúster y una base de datos de Azure Data Explorer mediante una plantilla de Azure Resource Manager

> [!div class="op_single_selector"]
> * [Portal](create-cluster-database-portal.md)
> * [CLI](create-cluster-database-cli.md)
> * [PowerShell](create-cluster-database-powershell.md)
> * [C#](create-cluster-database-csharp.md)
> * [Python](create-cluster-database-python.md)
> * [Plantilla de Azure Resource Manager](create-cluster-database-resource-manager.md)

El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro. Para usar Azure Data Explorer, cree primero un clúster y una o varias bases de datos en ese clúster. A continuación, ingerirá (cargará) los datos en una base de datos para que pueda ejecutar consultas en ella. 

En este artículo, crea un clúster y una base de datos de Azure Data Explorer mediante una [plantilla de Azure Resource Manager](/azure/azure-resource-manager/management/overview). El artículo muestra cómo definir los recursos que se implementan y los parámetros que se especifican cuando se ejecuta la implementación. Puede usar esta plantilla para sus propias implementaciones o personalizarla para satisfacer sus necesidades. Para información sobre la creación de plantillas, consulte [Creación de plantillas de Azure Resource Manager](/azure/azure-resource-manager/resource-group-authoring-templates). Para la sintaxis y las propiedades de JSON que se usan en una plantilla, consulte [Tipos de recursos de Microsoft.Kusto](/azure/templates/microsoft.kusto/allversions).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="azure-resource-manager-template-for-cluster-and-database-creation"></a>Plantilla de Azure Resource Manager para la creación de un clúster y una base de datos

En este inicio rápido se usa una [plantilla de inicio rápido existente](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-kusto-cluster-database/azuredeploy.json)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
      "clusters_kustocluster_name": {
          "type": "string",
          "defaultValue": "[concat('kusto', uniqueString(resourceGroup().id))]",
          "metadata": {
            "description": "Name of the cluster to create"
          }
      },
      "databases_kustodb_name": {
          "type": "string",
          "defaultValue": "kustodb",
          "metadata": {
            "description": "Name of the database to create"
          }
      },
      "location": {
        "type": "string",
        "defaultValue": "[resourceGroup().location]",
        "metadata": {
          "description": "Location for all resources."
        }
      }
  },
  "variables": {},
  "resources": [
      {
          "name": "[parameters('clusters_kustocluster_name')]",
          "type": "Microsoft.Kusto/clusters",
          "sku": {
              "name": "Standard_D13_v2",
              "tier": "Standard",
              "capacity": 2
          },
          "apiVersion": "2019-09-07",
          "location": "[parameters('location')]",
          "tags": {
            "Created By": "GitHub quickstart template"
          }
      },
      {
          "name": "[concat(parameters('clusters_kustocluster_name'), '/', parameters('databases_kustodb_name'))]",
          "type": "Microsoft.Kusto/clusters/databases",
          "apiVersion": "2019-09-07",
          "location": "[parameters('location')]",
          "dependsOn": [
              "[resourceId('Microsoft.Kusto/clusters', parameters('clusters_kustocluster_name'))]"
          ],
          "properties": {
              "softDeletePeriodInDays": 365,
              "hotCachePeriodInDays": 31
          }
      }
  ]
}
```

Para buscar más ejemplos de plantillas, consulte [Plantillas de inicio rápido de Azure](https://azure.microsoft.com/resources/templates/).

## <a name="deploy-the-template-and-verify-template-deployment"></a>Implementación de la plantilla y comprobación de esta

Puede implementar la plantilla de Azure Resource Manager [mediante Azure Portal](#use-the-azure-portal-to-deploy-the-template-and-verify-template-deployment) o [mediante PowerShell](#use-powershell-to-deploy-the-template-and-verify-template-deployment).

### <a name="use-the-azure-portal-to-deploy-the-template-and-verify-template-deployment"></a>Uso de Azure Portal para implementar la plantilla y comprobación de esta

1. Con el fin de crear un clúster y una base de datos, use el botón siguiente para iniciar la implementación. Haga clic con el botón derecho y seleccione **Abrir en una ventana nueva** para seguir el resto de los pasos de este artículo.

    [![Implementación en Azure](media/create-cluster-database-resource-manager/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-kusto-cluster-database%2Fazuredeploy.json)

    El botón **Deploy to Azure** (Implementar en Azure) le lleva a Azure Portal para rellenar un formulario de implementación.

    ![Implementar en Azure](media/create-cluster-database-resource-manager/deploy-2-azure.png)

    Puede [editar e implementar la plantilla en Azure Portal](/azure/azure-resource-manager/resource-manager-quickstart-create-templates-use-the-portal#edit-and-deploy-the-template) mediante el formulario.

1. Complete las secciones **ASPECTOS BÁSICOS** y **CONFIGURACIÓN**. Seleccione nombres de base de datos y clúster únicos.
La creación de un clúster y una base de datos de Azure Data Explorer tarda unos minutos.

1. Para comprobar la implementación, abra el grupo de recursos en [Azure Portal](https://portal.azure.com) para encontrar sus nuevos clúster y base de datos. 

### <a name="use-powershell-to-deploy-the-template-and-verify-template-deployment"></a>Uso de PowerShell para implementar la plantilla y comprobación de esta

#### <a name="deploy-the-template-using-powershell"></a>Implementación de la plantilla mediante PowerShell

1. Seleccione **Probar** en el bloque de código siguiente y luego siga las instrucciones para iniciar sesión en Azure Cloud Shell.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $resourceGroupName = "${projectName}rg"
    $clusterName = "${projectName}cluster"
    $parameters = @{}
    $parameters.Add("clusters_kustocluster_name", $clusterName)
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-kusto-cluster-database/azuredeploy.json"
    New-AzResourceGroup -Name $resourceGroupName -Location $location
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -TemplateParameterObject $parameters
    Write-Host "Press [ENTER] to continue ..."
    ```

1. Seleccione **Copiar** para copiar el script de PowerShell.
1. Haga clic con el botón derecho en la consola del shell y seleccione **Pegar**.
La creación de un clúster y una base de datos de Azure Data Explorer tarda unos minutos.

#### <a name="verify-the-deployment-using-powershell"></a>Comprobación de la implementación mediante PowerShell

Para comprobar la implementación, use el siguiente script de Azure PowerShell.  Si Cloud Shell sigue abierto, no es necesario copiar ni ejecutar la primera línea (Read-Host). Para obtener más información sobre la administración de recursos de Azure Data Explorer en PowerShell, lea [Az.Kusto](/powershell/module/az.kusto/?view=azps-2.7.0). 

```azurepowershell-interactive
$projectName = Read-Host -Prompt "Enter the same project name that you used in the last procedure"

Install-Module -Name Az.Kusto
$resourceGroupName = "${projectName}rg"
$clusterName = "${projectName}cluster"

Get-AzKustoCluster -ResourceGroupName $resourceGroupName -Name $clusterName
Write-Host "Press [ENTER] to continue ..."
```

[!INCLUDE [data-explorer-clean-resources](includes/data-explorer-clean-resources.md)]

## <a name="next-steps"></a>Pasos siguientes

[Ingest data into Azure Data Explorer cluster and database](ingest-data-overview.md) (Ingesta de datos en el clúster y la base de datos de Azure Data Explorer)
