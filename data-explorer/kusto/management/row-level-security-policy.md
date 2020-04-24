---
title: 'Directiva RowLevelSecurity (versión preliminar): Explorador de azure Data Explorer ( RowLevelSecurity policy ( versión preliminar) - Explorador de datos de Azure Microsoft Docs'
description: En este artículo se describe la directiva RowLevelSecurity (versión preliminar) en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/25/2020
ms.openlocfilehash: bf8ee8bc4c43c4ed3bcd320ce5b4778f33c3cc64
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520305"
---
# <a name="rowlevelsecurity-policy-preview"></a>Directiva RowLevelSecurity (versión preliminar)

En este artículo se describen los comandos utilizados para configurar una [directiva de row_level_security](rowlevelsecuritypolicy.md) para tablas de base de datos.

## <a name="displaying-the-policy"></a>Visualización de la política

Para mostrar la directiva, utilice el siguiente comando:

```kusto
.show table <table_name> policy row_level_security
```

## <a name="configuring-the-policy"></a>Configuración de la directiva

Habilite row_level_security directiva en una tabla:

```kusto
.alter table <table_name> policy row_level_security enable "<query>"
```

Deshabilite row_level_security directiva en una tabla:

```kusto
.alter table <table_name> policy row_level_security disable "<query>"
```

Incluso cuando la directiva está deshabilitada, puede forzarla a aplicarla a una sola consulta. Introduzca la siguiente línea antes de la consulta:

`set query_force_row_level_security;`

Esto es útil si desea probar varias consultas para row_level_security, pero no desea que realmente surta efecto en los usuarios. Una vez que esté seguro de la consulta, habilite la directiva.

> [!NOTE]
> Las siguientes restricciones `query`se aplican a:
>
> * La consulta debe producir exactamente el mismo esquema que la tabla en la que se define la directiva. Es decir, el resultado de la consulta debe devolver exactamente las mismas columnas que la tabla original, en el mismo orden, con los mismos nombres y tipos.
> * La consulta solo puede utilizar `extend` `where`los `project` `project-away`siguientes `project-rename` `project-reorder` operadores: , , , , , y `union`.
> * La consulta no puede hacer referencia a tablas distintas de la que se define la directiva.
> * La consulta puede ser cualquiera de las siguientes o una combinación de ellas:
>    * Consulta (por `<table_name> | extend CreditCardNumber = "****"`ejemplo, )
>    * Función (por `AnonymizeSensitiveData`ejemplo, )
>    * Datatable (por `datatable(Col1:datetime, Col2:string) [...]`ejemplo, )

> [!TIP]
> Estas funciones suelen ser útiles para consultas row_level_security:
> * [current_principal()](../query/current-principalfunction.md)
> * [current_principal_details()](../query/current-principal-detailsfunction.md)
> * [current_principal_is_member_of()](../query/current-principal-ismemberoffunction.md)

**Ejemplo**

```kusto
.create-or-alter function with () TrimCreditCardNumbers() {
    let UserCanSeeFullNumbers = current_principal_is_member_of('aadgroup=super_group@domain.com');
    let AllData = Customers | where UserCanSeeFullNumbers;
    let PartialData = Customers | where not(UserCanSeeFullNumbers) | extend CreditCardNumber = "****";
    union AllData, PartialData
}

.alter table Customers policy row_level_security enable "TrimCreditCardNumbers"
```
**Nota**de `UserCanSeeFullNumbers` rendimiento : se evaluará `AllData` primero, y luego se evaluará o `PartialData` no ambos, que es el resultado esperado.
Puede leer más sobre el impacto en el rendimiento de RLS [aquí](rowlevelsecuritypolicy.md#performance-impact-on-queries).

## <a name="deleting-the-policy"></a>Eliminación de la política

```kusto
.delete table <table_name> policy row_level_security
```