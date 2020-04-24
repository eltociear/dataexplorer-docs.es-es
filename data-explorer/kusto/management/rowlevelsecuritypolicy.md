---
title: 'Seguridad de nivel de fila (versión preliminar): Explorador de azure data Explorer ( Row Level Security) - Explorador de datos de Azure Microsoft Docs'
description: En este artículo se describe la seguridad de nivel de fila (versión preliminar) en el Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/25/2020
ms.openlocfilehash: 548b0e9e40cfd709b40e548fe7e0a8ad42aa4abc
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520288"
---
# <a name="row-level-security-preview"></a>Seguridad de nivel de fila (versión preliminar)

Use el contexto de pertenencia o ejecución de grupo para controlar el acceso a las filas de una tabla de base de datos.

La seguridad de nivel de fila (RLS) simplifica el diseño y la codificación de la seguridad en la aplicación al permitirle aplicar restricciones en el acceso a filas de datos. Por ejemplo, limite el acceso de los usuarios a las filas relevantes para su departamento o restrinja el acceso de los clientes solo a los datos relevantes para su empresa.

La lógica de restricción de acceso se encuentra en el nivel de base de datos, en lugar de alejarse de los datos de otro nivel de aplicación. El sistema de base de datos aplica las restricciones de acceso cada vez que se intenta el acceso a los datos desde cualquier nivel. Esto hace que el sistema de seguridad resulte más sólido y confiable al reducir el área expuesta del sistema de seguridad.

RLS le permite proporcionar acceso a otras aplicaciones y/o usuarios solo a una determinada parte de una tabla. Por ejemplo, podría:

* Conceda acceso solo a filas que cumplan algunos criterios
* Anonimizar datos en algunas de las columnas
* Ambos de los anteriores

> [!NOTE]
> La directiva de seguridad de nivel de fila no se puede habilitar en una tabla:
> * Para el que está configurada [la exportación continua de datos.](../management/data-export/continuous-data-export.md)
> * A esto hace referencia una consulta de alguna directiva de [actualización](./updatepolicy.md).
> * En el que se configura la directiva de acceso de [vista restringida.](./restrictedviewaccesspolicy.md)

Para obtener más información sobre los comandos de control para administrar la directiva de seguridad de nivel de fila, [consulte aquí](../management/row-level-security-policy.md).

## <a name="examples"></a>Ejemplos

En una `Sales`tabla denominada , cada fila contiene detalles sobre una venta. Una de las columnas contiene el nombre del vendedor.
En lugar de dar a los `Sales`usuarios de ventas acceso a todos los registros en , puede habilitar una directiva de seguridad de nivel de fila en esta tabla para devolver solo los registros donde el vendedor es el usuario actual:

```kusto
Sales | where SalesPersonAadUser == current_principal()
```

También puede enmascarar el número de la tarjeta de crédito:

```kusto
Sales | where SalesPersonAadUser == current_principal() | extend CreditCardNumber = "****"
```

Si desea que cada vendedor vea todas las ventas de un país específico, puede definir una consulta similar a la siguiente:

```kusto
let UserToCountryMapping = datatable(User:string, Country:string)
[
  "john@domain.com", "USA",
  "anna@domain.com", "France"
];
Sales
| where Country in (UserToCountryMapping | where User == current_principal_details()["UserPrincipalName"] | project Country)
```

Si tiene un grupo de AAD que contiene los administradores de los vendedores, es posible que desee que tengan acceso a todas las filas. Esto se puede lograr mediante la siguiente consulta en la directiva de seguridad de nivel de fila:

```kusto
let IsManager = current_principal_is_member_of('aadgroup=sales_managers@domain.com');
let AllData = Sales | where IsManager;
let PartialData = Sales | where not(IsManager) and (SalesPersonAadUser == current_principal());
union AllData, PartialData
| extend CreditCardNumber = "****"
```

En general, si tiene varios grupos de AAD y desea que los miembros de cada grupo vean un subconjunto diferente de datos, puede seguir esta estructura para una consulta RLS (suponiendo que un usuario solo puede pertenecer a un único grupo de AAD):

```kusto
let IsInGroup1 = current_principal_is_member_of('aadgroup=group1@domain.com');
let IsInGroup2 = current_principal_is_member_of('aadgroup=group2@domain.com');
let IsInGroup3 = current_principal_is_member_of('aadgroup=group3@domain.com');
let DataForGroup1 = Customers | where IsInGroup1 and <filtering specific for group1>;
let DataForGroup2 = Customers | where IsInGroup2 and <filtering specific for group2>;
let DataForGroup3 = Customers | where IsInGroup3 and <filtering specific for group3>;
union DataForGroup1, DataForGroup2, DataForGroup3
```

## <a name="more-use-cases"></a>Más casos de uso

* Una persona de apoyo del centro de llamadas puede identificar a las personas que llaman por varios dígitos de su número de seguro social o número de tarjeta de crédito. Esos números no deben estar completamente expuestos a la persona de apoyo. Una directiva RLS se puede aplicar en la tabla para enmascarar todos excepto los últimos cuatro dígitos de cualquier seguro social o número de tarjeta de crédito en el conjunto de resultados de cualquier consulta.
* Establezca una directiva de RLS que enmascare la información de identificación personal (PII), lo que permite a los desarrolladores consultar entornos de producción con fines de solución de problemas sin infringir las normativas de cumplimiento.
* Un hospital puede establecer una política de RLS que permita a las enfermeras ver filas de datos solo para sus pacientes.
* Un banco puede establecer una directiva RLS para restringir el acceso a las filas de datos financieros en función de la división o el rol de negocio de un empleado.
* Una aplicación multiinquilino puede almacenar datos de muchos inquilinos en un único conjunto de tablas (que es muy eficaz). Usarían una directiva RLS para aplicar una separación lógica de las filas de datos de cada inquilino de las filas de cada otro inquilino, de modo que cada inquilino pueda ver solo sus filas de datos.

## <a name="performance-impact-on-queries"></a>Impacto en el rendimiento de las consultas

Cuando se habilita una directiva RLS en una tabla, habrá algún impacto en el rendimiento en las consultas que tienen acceso a esa tabla. El acceso a la tabla se reemplazará realmente por la consulta RLS que se define en esa tabla. El impacto en el rendimiento de una consulta RLS normalmente constará de dos partes:

* Comprobaciones de pertenencia en Azure Active Directory
* Los filtros aplicados a los datos

Por ejemplo:

```kusto
let IsRestrictedUser = current_principal_is_member_of('aadgroup=some_group@domain.com');
let AllData = MyTable | where not(IsRestrictedUser);
let PartialData = MyTable | where IsRestrictedUser and (...);
union AllData, PartialData
```

Si el usuario no some_group@domain.comforma `IsRestrictedUser` parte de `false`, entonces se evaluará en , por lo que la consulta que se evaluará será similar a esta:

```kusto
let AllData = MyTable;           // the condition evaluates to `true`, so the filter is dropped
let PartialData = <empty table>; // the condition evaluates to `false`, so the whole expression is replaced with an empty table
union AllData, PartialData       // this will just return AllData, as PartialData is empty
```

De forma `IsRestrictedUser` similar, `true`si se evalúa `PartialData` como , solo se evaluará la consulta para .

## <a name="performance-impact-on-ingestion"></a>Impacto en el rendimiento de la ingesta

Ninguno.