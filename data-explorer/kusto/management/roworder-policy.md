---
title: 'Directiva de RowOrder: Azure Explorador de datos'
description: En este artículo se describe la Directiva RowOrder en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/24/2020
ms.openlocfilehash: 63aad71854c73a3d1f1837c3665a152db8b48d13
ms.sourcegitcommit: 9fe6e34ef3321390ee4e366819ebc9b132b3e03f
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/01/2020
ms.locfileid: "84258035"
---
# <a name="roworder-policy"></a>Directiva RowOrder

En este artículo se describen los comandos de control que se usan para crear y modificar la [Directiva de orden de filas](../management/roworderpolicy.md).

## <a name="show-roworder-policy"></a>Mostrar Directiva de RowOrder

```kusto
.show table <table_name> policy roworder

.show table * policy roworder
```

## <a name="delete-roworder-policy"></a>Eliminar Directiva de RowOrder

```kusto
.delete table <table_name> policy roworder
```

## <a name="alter-roworder-policy"></a>Modificar Directiva de RowOrder

```kusto
.alter table <table_name> policy roworder (<row_order_policy>)

.alter tables (<table_name> [, ...]) policy roworder (<row_order_policy>)

.alter-merge table <table_name> policy roworder (<row_order_policy>)
```

**Ejemplos** 

En el ejemplo siguiente se establece la Directiva de orden de filas en la `TenantId` columna (ascendente) como clave principal y en la `Timestamp` columna (ascendente) como clave secundaria. A continuación, se consulta la Directiva.

```kusto
.alter table events policy roworder (TenantId asc, Timestamp desc)

.alter tables (events1, events2, events3) policy roworder (TenantId asc, Timestamp desc)

.show table events policy roworder 
```

|TableName|RowOrderPolicy| 
|---|---|
|events|(TenantId ASC, TIMESTAMP DESC)|
