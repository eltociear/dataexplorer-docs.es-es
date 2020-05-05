---
title: 'Instrucción de alias: Explorador de datos de Azure'
description: En este artículo se describe la instrucción de alias en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 63c639fb95322c537c5e069aa7e8ef7037371c88
ms.sourcegitcommit: 4f68d6dbfa6463dbb284de0aa17fc193d529ce3a
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/04/2020
ms.locfileid: "82742020"
---
# <a name="alias-statement"></a>Instrucción Alias

::: zone pivot="azuredataexplorer"

Las instrucciones de alias permiten definir un alias para las bases de datos, que se pueden usar más adelante en la misma consulta.

Esto resulta útil cuando se trabaja con varios clústeres pero se desea que aparezcan como si se estuviera trabajando en menos clústeres.
El alias debe definirse según la sintaxis siguiente, donde *ClusterName* y *DatabaseName* son entidades existentes y válidas.

**Sintaxis**

`alias`base de datos [*' DatabaseAliasName '*] `=` clúster ("https://*ClusterName*. kusto. Windows. net: 443"). base de datos ("*DatabaseName*")

`alias`clúster de *DatabaseAliasName* `=` de base de datos ("https://*ClusterName*. kusto. Windows. net: 443"). Database ("*DatabaseName*")

* *' DatabaseAliasName '* puede ser un nombre existente o un nombre nuevo.
* El URI del clúster asignado y el nombre de la base de datos asignada deben aparecer entre comillas dobles (") o comillas simples (')

**Ejemplos**

```kusto
alias database["wiki"] = cluster("https://somecluster.kusto.windows.net:443").database("somedatabase");
database("wiki").PageViews | count 
```

```kusto
alias database Logs = cluster("https://othercluster.kusto.windows.net:443").database("otherdatabase");
database("Logs").Traces | count 
```

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end
