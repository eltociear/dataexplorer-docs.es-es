---
title: 'Administración de roles de seguridad: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la administración de roles de seguridad en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: ea56911ff2ad320d1070da2a4b92d94f060273cf
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81520101"
---
# <a name="security-roles-management"></a>Gestión de roles de seguridad

> [!IMPORTANT]
> Antes de modificar las reglas de autorización en los clústeres de Kusto, lea lo siguiente: [Autorización](../management/access-control/index.md) 
> [basada en roles de](../management/access-control/role-based-authorization.md) visión general de control de acceso 

En este artículo se describen los comandos de control utilizados para administrar roles de seguridad.
Los roles de seguridad definen qué entidades de seguridad (usuarios y aplicaciones) tienen permisos para operar en un recurso protegido, como una base de datos o una tabla, y qué operaciones se permiten. Por ejemplo, las entidades de seguridad que tienen el `database viewer` rol de seguridad para una base de datos específica pueden consultar y ver todas las entidades de esa base de datos (con la excepción de las tablas restringidas).

El rol de seguridad se puede asociar con entidades de seguridad o grupos de seguridad (que pueden tener otras entidades de seguridad u otros grupos de seguridad). Cuando una entidad de seguridad intenta realizar una operación en un recurso protegido, el sistema comprueba que la entidad de seguridad está asociada a al menos un rol de seguridad que concede permisos para realizar esta operación en el recurso. Esto se denomina verificación de **autorización.** Si se produce un error en la comprobación de autorización, se anula la operación.

**Sintaxis**

Sintaxis de los comandos de administración de roles de seguridad:

*Verb* *SecurableObjectType* *SecurableObjectName* *Role* [`(` *ListOfPrincipals* `)` [*Description*]]

* *Verbo* indica el tipo de `.show`acción `.add` `.drop`que `.set`se debe realizar: , , , y .

    |*Verbo* |Descripción                                  |
    |-------|---------------------------------------------|
    |`.show`|Devuelve el valor o los valores actuales.         |
    |`.add` |Agrega una o más entidades de seguridad al rol.     |
    |`.drop`|Quita una o varias entidades de seguridad del rol.|
    |`.set` |Establece el rol en la lista específica de entidades de seguridad, eliminando todas las anteriores (si las hay).|

* *SecurableObjectType* es el tipo de objeto cuyo rol se especifica.

    |*SecurableObjectType*|Descripción|
    |---------------------|-----------|
    |`database`|La base de datos especificada|
    |`table`|La tabla especificada|

* *SecurableObjectName* es el nombre del objeto.

* *Rol* es el nombre del rol relevante.

    |*Rol*      |Descripción|
    |------------|-----------|
    |`principals`|Solo puede aparecer como `.show` parte de un verbo; devuelve la lista de entidades de seguridad que pueden afectar al objeto protegible.|
    |`admins`    |Tener control sobre el objeto protegible, incluida la capacidad de verlo, modificarlo y eliminar el objeto y todos los subobjetos.|
    |`users`     |Puede ver el objeto protegible y crear nuevos objetos debajo de él.|
    |`viewers`   |Puede ver el objeto protegible.|
    |`unrestrictedviewers`|Sólo en el nivel de base de datos, permite la `viewers` `users`visualización de tablas restringidas (que no están expuestas a "normal" y ).|
    |`ingestors` |Solo en el nivel de base de datos, permita la ingesta de datos en todas las tablas.|
    |`monitors`  ||

* *ListOfPrincipals* es una lista opcional delimitada por comas de `string`identificadores principales de seguridad (valores de tipo ).

* *Descripción* es un `string` valor opcional de tipo que se almacena junto con la asociación, para fines de auditoría futuros.

## <a name="example"></a>Ejemplo

El siguiente comando de control enumera todas las `StormEvents` entidades de seguridad que tienen acceso a la tabla en la base de datos:

```kusto
.show table StormEvents principals
```

Aquí están los resultados potenciales de este comando:

|Role |PrincipalType |PrincipalDisplayName |PrincipalObjectId |PrincipalFQN 
|---|---|---|---|---
|Administrador de Apsty de base de datos |Usuario de AAD |Mark Smith |cd709aed-a26c-e3953dec735e |adusermsmith@fabrikam.com|





## <a name="managing-database-security-roles"></a>Gestión de roles de seguridad de bases de datos

`.set``database` `none` `skip-results` *Rol* *DatabaseName* [ ]

`.set``database` *DatabaseName* *Role* `(` `,` *Principal* [ *Principal*...] `)` [`skip-results`] [*Descripción*]

`.add``database` *DatabaseName* *Role* `(` `,` *Principal* [ *Principal*...] `)` [`skip-results`] [*Descripción*]

`.drop``database` *DatabaseName* *Role* `(` `,` *Principal* [ *Principal*...] `)` [`skip-results`] [*Descripción*]

El primer comando quita todas las entidades de seguridad del rol. El segundo quita todas las entidades de seguridad del rol y establece un nuevo conjunto de entidades de seguridad. El tercero agrega nuevas entidades de seguridad al rol sin quitar las entidades de seguridad existentes. El último quita las entidades de seguridad indicadas de los roles y mantiene las demás.

Donde:

* *DatabaseName* es el nombre de la base de datos cuyo rol de seguridad se está modificando.

* *El* rol `admins` `ingestors`es: , `monitors`, , `unrestrictedviewers`, , `users`, o `viewers`.

* *El director* es uno o más directores. Consulte [entidades de seguridad y proveedores](./access-control/principals-and-identity-providers.md) de identidades para obtener información sobre cómo especificar estas entidades de seguridad.

* `skip-results`, si se proporciona, solicita que el comando no devuelva la lista actualizada de entidades de seguridad de base de datos.

* *Descripción*, si se proporciona, es texto que se `.show` asociará con el cambio y se recuperará mediante el comando correspondiente.

<!-- TODO: Need more examples for the public syntax. Until then we're keeping this internal -->


## <a name="managing-table-security-roles"></a>Gestión de roles de seguridad de tablas

`.set``table` `none` `skip-results` *Rol* *TableName* [ ]

`.set``table` *TableName* *Role* `(` `,` *Principal* [ *Principal*...] `)` [`skip-results`] [*Descripción*]

`.add``table` *TableName* *Role* `(` `,` *Principal* [ *Principal*...] `)` [`skip-results`] [*Descripción*]

`.drop``table` *TableName* *Role* `(` `,` *Principal* [ *Principal*...] `)` [`skip-results`] [*Descripción*]

El primer comando quita todas las entidades de seguridad del rol. El segundo quita todas las entidades de seguridad del rol y establece un nuevo conjunto de entidades de seguridad. El tercero agrega nuevas entidades de seguridad al rol sin quitar las entidades de seguridad existentes. El último quita las entidades de seguridad indicadas de los roles y mantiene las demás.

Donde:

* *TableName* es el nombre de la tabla cuyo rol de seguridad se está modificando.

* *El* rol `admins` `ingestors`es: o .

* *El director* es uno o más directores. Consulte [entidades de seguridad y proveedores](./access-control/principals-and-identity-providers.md) de identidades para obtener información sobre cómo especificar estas entidades de seguridad.

* `skip-results`, si se proporciona, solicita que el comando no devuelva la lista actualizada de entidades de seguridad de tabla.

* *Descripción*, si se proporciona, es texto que se `.show` asociará con el cambio y se recuperará mediante el comando correspondiente.

**Ejemplo**

```kusto
.add table Test admins ('aaduser=imike@fabrikam.com ')
```

## <a name="managing-function-security-roles"></a>Gestión de roles de seguridad de funciones

`.set``function` `none` `skip-results` *Rol* *FunctionName* [ ]

`.set``function` *FunctionName* *Role* `(` `,` *Principal* [ *Principal*...] `)` [`skip-results`] [*Descripción*]

`.add``function` *FunctionName* *Role* `(` `,` *Principal* [ *Principal*...] `)` [`skip-results`] [*Descripción*]

`.drop``function` *FunctionName* *Role* `(` `,` *Principal* [ *Principal*...] `)` [`skip-results`] [*Descripción*]

El primer comando quita todas las entidades de seguridad del rol. El segundo quita todas las entidades de seguridad del rol y establece un nuevo conjunto de entidades de seguridad. El tercero agrega nuevas entidades de seguridad al rol sin quitar las entidades de seguridad existentes. El último quita las entidades de seguridad indicadas de los roles y mantiene las demás.

Donde:

* *FunctionName* es el nombre de la función cuyo rol de seguridad se está modificando.

* *El* rol `admin`siempre es .

* *El director* es uno o más directores. Consulte [entidades de seguridad y proveedores](./access-control/principals-and-identity-providers.md) de identidades para obtener información sobre cómo especificar estas entidades de seguridad.

* `skip-results`, si se proporciona, solicita que el comando no devuelva la lista actualizada de entidades de seguridad de función.

* *Descripción*, si se proporciona, es texto que se `.show` asociará con el cambio y se recuperará mediante el comando correspondiente.

**Ejemplo**

```kusto
.add function MyFunction admins ('aaduser=imike@fabrikam.com') 'This user should have access'
```

