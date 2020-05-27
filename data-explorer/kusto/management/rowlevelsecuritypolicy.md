---
title: 'Seguridad de nivel de fila (versión preliminar): Explorador de datos de Azure | Microsoft Docs'
description: En este artículo se describe Seguridad de nivel de fila (versión preliminar) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/25/2020
ms.openlocfilehash: 2d535e47f5b05c1c45ef2cf1993681aa8aa2133d
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83863309"
---
# <a name="row-level-security-preview"></a>Seguridad de nivel de fila (vista previa)

Utilice la pertenencia a grupos o contexto de ejecución para controlar el acceso a las filas de una tabla de base de datos.

Seguridad de nivel de fila (RLS) simplifica el diseño y la codificación de la seguridad en la aplicación, ya que permite aplicar restricciones en el acceso a las filas de datos. Por ejemplo, limite el acceso de los usuarios a las filas relevantes para su departamento o restrinja el acceso de los clientes solo a los datos relevantes para su empresa.

La lógica de restricción de acceso se encuentra en el nivel de base de datos, en lugar de fuera de los datos de otra capa de aplicación. El sistema de base de datos aplica las restricciones de acceso cada vez que se intenta acceder a los datos desde cualquier nivel. Esto hace que el sistema de seguridad resulte más sólido y confiable al reducir el área expuesta del sistema de seguridad.

RLS le permite proporcionar acceso a otras aplicaciones y/o usuarios solo a una determinada parte de una tabla. Por ejemplo, podría:

* Conceder acceso solo a las filas que cumplen algunos criterios
* Anonimización datos en algunas de las columnas
* Ambos de los anteriores

Para obtener más información, vea [comandos de control para administrar la Directiva de seguridad de nivel de fila](../management/row-level-security-policy.md).

> [!Note]
> La Directiva RLS que configure en la base de datos de producción también tendrá efecto en las bases de datos de seguimiento. No puede configurar diferentes directivas de RLS en las bases de datos de seguimiento y de producción.

## <a name="limitations"></a>Limitaciones

No hay ningún límite en el número de tablas en las que se puede configurar la Directiva de Seguridad de nivel de fila.

No se puede habilitar la Directiva RLS en una tabla:
* Para el que se ha configurado la [exportación continua de datos](../management/data-export/continuous-data-export.md) .
* A la que hace referencia una consulta de alguna [Directiva de actualización](./updatepolicy.md).
* En la que está configurada la [Directiva de acceso de vista restringida](./restrictedviewaccesspolicy.md) .

## <a name="examples"></a>Ejemplos

### <a name="limiting-access-to-sales-table"></a>Limitar el acceso a la tabla sales

En una tabla denominada `Sales` , cada fila contiene detalles sobre una venta. Una de las columnas contiene el nombre del vendedor. En lugar de ofrecer a los usuarios de ventas acceso a todos los registros de `Sales` , puede habilitar una directiva de seguridad de nivel de fila en esta tabla para devolver solo los registros en los que el vendedor es el usuario actual:

```kusto
Sales | where SalesPersonAadUser == current_principal()
```

También puede enmascarar el número de la tarjeta de crédito:

```kusto
Sales | where SalesPersonAadUser == current_principal() | extend CreditCardNumber = "****"
```

Si desea que todos los vendedores vean todas las ventas de un país específico, puede definir una consulta similar a la siguiente:

```kusto
let UserToCountryMapping = datatable(User:string, Country:string)
[
  "john@domain.com", "USA",
  "anna@domain.com", "France"
];
Sales
| where Country in (UserToCountryMapping | where User == current_principal_details()["UserPrincipalName"] | project Country)
```

Si tiene un grupo de AAD que contiene los administradores de los vendedores, puede que desee tener acceso a todas las filas. Esto puede lograrse mediante la siguiente consulta en la Directiva de Seguridad de nivel de fila:

```kusto
let IsManager = current_principal_is_member_of('aadgroup=sales_managers@domain.com');
let AllData = Sales | where IsManager;
let PartialData = Sales | where not(IsManager) and (SalesPersonAadUser == current_principal());
union AllData, PartialData
| extend CreditCardNumber = "****"
```

### <a name="exposing-different-data-to-members-of-different-aad-groups"></a>Exposición de datos diferentes a miembros de grupos de AAD diferentes

Si tiene varios grupos de AAD y desea que los miembros de cada grupo vean un subconjunto de datos diferente, puede seguir esta estructura para una consulta de RLS (suponiendo que un usuario solo puede pertenecer a un único grupo de AAD):

```kusto
let IsInGroup1 = current_principal_is_member_of('aadgroup=group1@domain.com');
let IsInGroup2 = current_principal_is_member_of('aadgroup=group2@domain.com');
let IsInGroup3 = current_principal_is_member_of('aadgroup=group3@domain.com');
let DataForGroup1 = Customers | where IsInGroup1 and <filtering specific for group1>;
let DataForGroup2 = Customers | where IsInGroup2 and <filtering specific for group2>;
let DataForGroup3 = Customers | where IsInGroup3 and <filtering specific for group3>;
union DataForGroup1, DataForGroup2, DataForGroup3
```

### <a name="applying-the-same-rls-function-on-multiple-tables"></a>Aplicar la misma función RLS en varias tablas

En primer lugar, defina una función que reciba el nombre de tabla como un parámetro de cadena y haga referencia a la tabla mediante el `table()` operador. Por ejemplo:

```
.create-or-alter function RLSForCustomersTables(TableName: string) {
    table(TableName)
    | ...
}
```

A continuación, configure RLS en varias tablas de esta manera:

```
.alter table Customers1 policy row_level_security enable "RLSForCustomersTables('Customers1')"
.alter table Customers2 policy row_level_security enable "RLSForCustomersTables('Customers2')"
.alter table Customers3 policy row_level_security enable "RLSForCustomersTables('Customers3')"
```

## <a name="more-use-cases"></a>Más casos de uso

* Una persona de soporte técnico del centro de llamadas puede identificar a los autores de llamadas con varios dígitos de su número de la seguridad social o de una tarjeta de crédito. Estos números no deben exponerse por completo al personal de soporte técnico. Se puede aplicar una directiva de RLS en la tabla para enmascarar todos los dígitos, excepto los cuatro últimos, de cualquier número de la seguridad social o de una tarjeta de crédito en el conjunto de resultados de cualquier consulta.
* Establezca una directiva de RLS que oculte la información de identificación personal (PII), permitiendo a los desarrolladores consultar entornos de producción para solucionar problemas sin infringir las regulaciones de cumplimiento.
* Un hospital puede establecer una directiva de RLS que permita a los enfermeras ver solo las filas de datos de sus pacientes.
* Un banco puede establecer una directiva de RLS para restringir el acceso a las filas de datos financieros según la división de negocios o el rol de un empleado.
* Una aplicación de varios inquilinos puede almacenar datos de muchos inquilinos en un único tableset (que es muy eficaz). Usarían una directiva RLS para aplicar una separación lógica de las filas de datos de cada inquilino de las filas de los demás inquilinos, por lo que cada inquilino solo puede ver sus filas de datos.

## <a name="performance-impact-on-queries"></a>Impacto en el rendimiento de las consultas

Cuando se habilita una directiva RLS en una tabla, se producirá un impacto en el rendimiento de las consultas que tengan acceso a esa tabla. El acceso a la tabla se reemplazará realmente por la consulta RLS que se define en esa tabla. Normalmente, el impacto en el rendimiento de una consulta de RLS constará de dos partes:

* Comprobaciones de pertenencia en Azure Active Directory
* Filtros aplicados a los datos

Por ejemplo:

```kusto
let IsRestrictedUser = current_principal_is_member_of('aadgroup=some_group@domain.com');
let AllData = MyTable | where not(IsRestrictedUser);
let PartialData = MyTable | where IsRestrictedUser and (...);
union AllData, PartialData
```

Si el usuario no forma parte de some_group@domain.com , se `IsRestrictedUser` evaluará como `false` , de modo que la consulta que se va a evaluar será similar a esta:

```kusto
let AllData = MyTable;           // the condition evaluates to `true`, so the filter is dropped
let PartialData = <empty table>; // the condition evaluates to `false`, so the whole expression is replaced with an empty table
union AllData, PartialData       // this will just return AllData, as PartialData is empty
```

Del mismo modo, si se `IsRestrictedUser` evalúa como `true` , solo `PartialData` se evaluará la consulta para.

### <a name="improve-query-performance-when-rls-is-used"></a>Mejorar el rendimiento de las consultas cuando se utiliza RLS

* Si se aplica un filtro en una columna de alta cardinalidad (por ejemplo, DeviceID), considere la posibilidad de usar la Directiva de [particionamiento](./partitioningpolicy.md) o la [Directiva de orden de filas](./roworderpolicy.md) .
* Si se aplica un filtro en una columna de cardinalidad de bajo nivel, considere la posibilidad de usar la [Directiva de orden de filas](./roworderpolicy.md)

## <a name="performance-impact-on-ingestion"></a>Impacto en el rendimiento de la ingesta

No se produce ningún impacto en el rendimiento de la ingesta.
