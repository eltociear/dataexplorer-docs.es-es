---
title: 'Directiva de RowLevelSecurity (versión preliminar): Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la Directiva de RowLevelSecurity (versión preliminar) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/25/2020
ms.openlocfilehash: 3f36fa4c35ceb88c82b4dfcb7557e4839fed4aa2
ms.sourcegitcommit: 8e097319ea989661e1958efaa1586459d2b69292
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/15/2020
ms.locfileid: "84780548"
---
# <a name="rowlevelsecurity-policy-preview"></a>Directiva RowLevelSecurity (versión preliminar)

En este artículo se describen los comandos utilizados para configurar una [Directiva de row_level_security](rowlevelsecuritypolicy.md) para las tablas de base de datos.

## <a name="displaying-the-policy"></a>Mostrar la Directiva

Para mostrar la Directiva, use el siguiente comando:

```kusto
.show table <table_name> policy row_level_security
```

## <a name="configuring-the-policy"></a>Configuración de la Directiva

Habilitar la Directiva de row_level_security en una tabla:

```kusto
.alter table <table_name> policy row_level_security enable "<query>"
```

Deshabilitar la Directiva de row_level_security en una tabla:

```kusto
.alter table <table_name> policy row_level_security disable "<query>"
```

Incluso cuando la Directiva está deshabilitada, puede forzarla para que se aplique a una sola consulta. Escriba la siguiente línea antes de la consulta:

`set query_force_row_level_security;`

Esto resulta útil si desea probar varias consultas para row_level_security, pero no desea que se aplique realmente a los usuarios. Una vez que esté seguro en la consulta, habilite la Directiva.

> [!NOTE]
> Las restricciones siguientes se aplican a `query` :
>
> * La consulta debe generar exactamente el mismo esquema que la tabla en la que se define la Directiva. Es decir, el resultado de la consulta debe devolver exactamente las mismas columnas que la tabla original, en el mismo orden, con los mismos nombres y tipos.
> * La consulta solo puede utilizar los operadores siguientes: `extend` , `where` , `project` , `project-away` , `project-rename` , `project-reorder` `join` y `union` .
> * La consulta no puede hacer referencia a otras tablas en las que está habilitado RLS.
> * La consulta puede ser cualquiera de las siguientes, o una combinación de ellas:
>    * Consulta (por ejemplo, `<table_name> | extend CreditCardNumber = "****"` )
>    * Función (por ejemplo, `AnonymizeSensitiveData` )
>    * DataTable (por ejemplo, `datatable(Col1:datetime, Col2:string) [...]` )

> [!TIP]
> Estas funciones suelen ser útiles para las consultas de row_level_security:
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

**Nota de rendimiento**: se `UserCanSeeFullNumbers` evaluará primero y, a continuación, se `AllData` `PartialData` evaluará o, pero no ambos, que es el resultado esperado.
Puede leer más sobre el impacto en el rendimiento de RLS [aquí](rowlevelsecuritypolicy.md#performance-impact-on-queries).

## <a name="deleting-the-policy"></a>Eliminación de la Directiva

```kusto
.delete table <table_name> policy row_level_security
```
