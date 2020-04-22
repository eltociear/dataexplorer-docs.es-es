---
title: 'Directiva de llamada: Explorador de datos de Azure Microsoft Docs'
description: En este artículo se describe la directiva de llamada en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
ms.openlocfilehash: df57c4901cef74d574108d2c6e75672d1faba75c
ms.sourcegitcommit: e94be7045d71a0435b4171ca3a7c30455e6dfa57
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81744516"
---
# <a name="callout-policy"></a>Directiva de llamada

## <a name="overview"></a>Información general

Los clústeres de Azure Data Explorer pueden comunicarse con servicios externos en muchos escenarios diferentes.
Los administradores de clústeres pueden administrar los dominios permitidos para llamadas externas actualizando la directiva de llamada del clúster.

Las directivas de llamada se administran en el nivel de clúster y se clasifican en los siguientes tipos:
* `kusto`: controla las consultas entre clústeres del Explorador de datos de Azure.
* `sql`- controla el [plugin SQL](../query/sqlrequestplugin.md).


* `webapi`- controla otras llamadas web externas.
* `sandbox_artifacts`- Controla plugins de sandboxed ([python](../query/pythonplugin.md) | [R](../query/rplugin.md)).

La política de llamada se compone de los siguientes componentes:
* **CalloutType** define el tipo de la llamada y puede ser uno de los siguientes: kusto, sql o webapi
* **CalloutUriRegex** especifica el Regex permitido del dominio de la llamada
* **CanCall** indica si la llamada es permite llamadas externas.

## <a name="predefined-callout-policies"></a>Políticas de llamada predefinidas

Hay un conjunto de directivas de llamada predefinidas inmutablemente preconfiguradas en todos los clústeres de Azure Data Explorer para facilitar las llamadas a determinados servicios.

|Servicio      |Nube        |Designación  |Dominios permitidos |
|-------------|-------------|-------------|-------------|
|Kusto |Public Azure (Azure público) |Consultas entre clústeres |`^[^.]*\.kusto\.windows\.net$` <br> `^[^.]*\.kustomfa\.windows\.net$` |
|Kusto |Selva Negra |Consultas entre clústeres |`^[^.]*\.kusto\.cloudapi\.de$` <br> `^[^.]*\.kustomfa\.cloudapi\.de$` |
|Kusto |Fairfax |Consultas entre clústeres |`^[^.]*\.kusto\.usgovcloudapi\.net$` <br> `^[^.]*\.kustomfa\.usgovcloudapi\.net$` |
|Kusto |Mooncake |Consultas entre clústeres |`^[^.]*\.kusto\.chinacloudapi\.cn$` <br> `^[^.]*\.kustomfa\.chinacloudapi\.cn$` |
|Azure DB |Public Azure (Azure público) |Solicitudes SQL |`^[^.]*\.database\.windows\.net$` <br> `^[^.]*\.databasemfa\.windows\.net$` |
|Azure DB |Selva Negra |Solicitudes SQL |`^[^.]*\.database\.cloudapi\.de$` <br> `^[^.]*\.databasemfa\.cloudapi\.de$` |
|Azure DB |Fairfax |Solicitudes SQL |`^[^.]*\.database\.usgovcloudapi\.net$` <br> `^[^.]*\.databasemfa\.usgovcloudapi\.net$` |
|Azure DB |Mooncake |Solicitudes SQL |`^[^.]*\.database\.chinacloudapi\.cn$` <br> `^[^.]*\.databasemfa\.chinacloudapi\.cn$` |
|Servicio de base |Public Azure (Azure público) |Solicitudes de base |`baseliningsvc-int.azurewebsites.net` <br> `baseliningsvc-ppe.azurewebsites.net` <br> `baseliningsvc-prod.azurewebsites.net` |


## <a name="control-commands"></a>Comandos de control

Los comandos requieren permisos [AllDatabasesAdmin.](access-control/role-based-authorization.md)

**Mostrar todas las directivas de llamada configuradas**

```kusto
.show cluster policy callout
```

**Modificar las políticas de llamada de salida**

```kusto
.alter cluster policy callout @'[{"CalloutType": "webapi","CalloutUriRegex": "en\\.wikipedia\\.org","CanCall": true}]'
```

**Añadir un conjunto de llamadas permitidas**

```kusto
.alter-merge cluster policy callout @'[{"CalloutType": "webapi","CalloutUriRegex": "en\\.wikipedia\\.org","CanCall": true}, {"CalloutType": "webapi","CalloutUriRegex": "bing\\.com","CanCall": true}]'
```

**Eliminar todas las políticas de llamada no inmutables**

```kusto
.delete cluster policy callout
```
