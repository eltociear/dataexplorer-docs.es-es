---
title: 'Directiva RowOrder: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la directiva RowOrder en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: cda4c9a6017071878832fab376a0376d250f3ed6
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520254"
---
# <a name="roworder-policy"></a>Directiva RowOrder

En este artículo se describen los comandos de control utilizados para crear y modificar la directiva de orden de [filas.](../management/roworderpolicy.md)

## <a name="show-roworder-policy"></a>Mostrar directiva RowOrder

```kusto
.show table <table_name> policy roworder

.show table * policy roworder
```

## <a name="delete-roworder-policy"></a>Eliminar directiva RowOrder

```kusto
.delete table <table_name> policy roworder
```

## <a name="alter-roworder-policy"></a>Política De Modificar RowOrder

```kusto
.alter table <table_name> policy roworder (<row_order_policy>)

.alter tables (<table_name> [, ...]) policy roworder (<row_order_policy>)

.alter-merge table <table_name> policy roworder (<row_order_policy>)
```

**Ejemplos**

En el ejemplo siguiente se establece `TenantId` la directiva de orden de filas para `Timestamp` que esté en la columna (ascendente) como una clave principal y en la columna (ascendente) como clave secundaria; a continuación, consulta la directiva:

```kusto
.alter table events policy roworder (TenantId asc, Timestamp desc)

.alter tables (events1, events2, events3) policy roworder (TenantId asc, Timestamp desc)

.show table events policy roworder 
```

|TableName|RowOrderPolicy| 
|---|---|
|events|(TenantId asc, Timestamp desc)| 