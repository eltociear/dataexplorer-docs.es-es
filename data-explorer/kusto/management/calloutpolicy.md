---
title: 'Directiva de llamada: Azure Explorador de datos'
description: En este artículo se describe la Directiva de llamada en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: 42254e00e629a19dfceeef2d4a6c2d1877400c05
ms.sourcegitcommit: 283cce0e7635a2d8ca77543f297a3345a5201395
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2020
ms.locfileid: "84011557"
---
# <a name="callout-policy"></a>Directiva de llamada

Los clústeres de Azure Explorador de datos pueden comunicarse con servicios externos en muchos escenarios diferentes.
Los administradores de clústeres pueden administrar los dominios autorizados para llamadas externas mediante la actualización de la Directiva de llamada del clúster.

Las directivas de llamada se administran en el nivel de clúster y se clasifican en los siguientes tipos.
* `kusto`: Controla las consultas entre clústeres de Azure Explorador de datos.
* `sql`: Controla el [complemento de SQL](../query/sqlrequestplugin.md).

* `webapi`-Controla otras llamadas web externas.
* `sandbox_artifacts`-Controla los complementos de espacio aislado ([Python](../query/pythonplugin.md)  |  [R](../query/rplugin.md)).

La Directiva de llamada se compone de lo siguiente.

* **CalloutType** : define el tipo de la llamada y puede ser `kusto` , `sql` o.`webapi`
* **CalloutUriRegex** : especifica el regex permitido del dominio de la llamada.
* **CanCall** : indica si la llamada es llamadas externas permitidas.

## <a name="predefined-callout-policies"></a>Directivas de llamada predefinidas

La tabla muestra un conjunto de directivas de llamada predefinidas que están preconfiguradas en todos los clústeres de Azure Explorador de datos para habilitar llamadas para seleccionar servicios.

|Servicio      |Nube        |Designación  |Dominios permitidos |
|-------------|-------------|-------------|-------------|
|Kusto |`Public Azure` |Consultas entre clústeres |`^[^.]*\.kusto\.windows\.net$` <br> `^[^.]*\.kustomfa\.windows\.net$` |
|Kusto |`Black Forest` |Consultas entre clústeres |`^[^.]*\.kusto\.cloudapi\.de$` <br> `^[^.]*\.kustomfa\.cloudapi\.de$` |
|Kusto |`Fairfax` |Consultas entre clústeres |`^[^.]*\.kusto\.usgovcloudapi\.net$` <br> `^[^.]*\.kustomfa\.usgovcloudapi\.net$` |
|Kusto |`Mooncake` |Consultas entre clústeres |`^[^.]*\.kusto\.chinacloudapi\.cn$` <br> `^[^.]*\.kustomfa\.chinacloudapi\.cn$` |
|BASE de Azure |`Public Azure` |Solicitudes SQL |`^[^.]*\.database\.windows\.net$` <br> `^[^.]*\.databasemfa\.windows\.net$` |
|BASE de Azure |`Black Forest` |Solicitudes SQL |`^[^.]*\.database\.cloudapi\.de$` <br> `^[^.]*\.databasemfa\.cloudapi\.de$` |
|BASE de Azure |`Fairfax` |Solicitudes SQL |`^[^.]*\.database\.usgovcloudapi\.net$` <br> `^[^.]*\.databasemfa\.usgovcloudapi\.net$` |
|BASE de Azure |`Mooncake` |Solicitudes SQL |`^[^.]*\.database\.chinacloudapi\.cn$` <br> `^[^.]*\.databasemfa\.chinacloudapi\.cn$` |
|Servicio de línea de vida |Public Azure (Azure público) |Solicitudes de línea de |`baseliningsvc-int.azurewebsites.net` <br> `baseliningsvc-ppe.azurewebsites.net` <br> `baseliningsvc-prod.azurewebsites.net` |

## <a name="control-commands"></a>Comandos de control

Los comandos requieren permisos [AllDatabasesAdmin](access-control/role-based-authorization.md) .

**Mostrar todas las directivas de llamada configuradas**

```kusto
.show cluster policy callout
```

**Modificar directivas de llamada**

```kusto
.alter cluster policy callout @'[{"CalloutType": "webapi","CalloutUriRegex": "en\\.wikipedia\\.org","CanCall": true}]'
```

**Agregar un conjunto de llamadas permitidas**

```kusto
.alter-merge cluster policy callout @'[{"CalloutType": "webapi","CalloutUriRegex": "en\\.wikipedia\\.org","CanCall": true}, {"CalloutType": "webapi","CalloutUriRegex": "bing\\.com","CanCall": true}]'
```

**Eliminar todas las directivas de llamada no inmutables**

```kusto
.delete cluster policy callout
```
