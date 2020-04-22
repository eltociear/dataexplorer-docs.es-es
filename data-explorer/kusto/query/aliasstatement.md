---
title: 'Instrucción alias: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la instrucción Alias en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: c6c689ab6daacebe1cd20742b199c8b9cc299245
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81766093"
---
# <a name="alias-statement"></a>Instrucción Alias

::: zone pivot="azuredataexplorer"

Las instrucciones Alias permiten definir alias para bases de datos que se pueden usar más adelante en la misma consulta.

Esto es útil cuando se trabaja con varios clústeres al intentar aparecer como trabajar en menos clústeres o solo en un clúster.
El alias debe definirse de acuerdo con la sintaxis siguiente, donde *clustername* y *databasename* deben ser entidades existentes y válidas.

**Sintaxis**

`alias`database[*'DatabaseAliasName'*] `=` cluster("https://*clustername*.kusto.windows.net:443").database("*databasename*")

`alias`*DatabaseAliasName* `=` cluster("https://*clustername*.kusto.windows.net:443").database("*databasename*")

* *'DatabaseAliasName'* puede ser en axisting name o un nuevo nombre.
* El cluster-uri asignado y el nombre de base de datos asignado deben aparecer dentro de double-quotes(") o single-quotes(')

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

Esto no se admite en Azure Monitor

::: zone-end
