---
title: 'Principales y proveedores de identidades: Explorador de datos de Azure Microsoft Docs'
description: En este artículo se describen las entidades de seguridad y los proveedores de identidades en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: a0638ba0031162dadbb4b9a2815940e66d4dcfb3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522651"
---
# <a name="principals-and-identity-providers"></a>Directores y proveedores de identidades

El modelo de autorización de Kusto admite varios proveedores de identidades (IdP) y varios tipos principales.
En este artículo se revisan los tipos principales admitidos y se muestra su uso con comandos de asignación de [roles.](../../management/security-roles.md)

### <a name="azure-active-directory"></a>Azure Active Directory
Azure Active Directory (AAD) es el proveedor de identidades y servicio de directorio en la nube multiinquilino preferido de Azure, capaz de autenticar entidades de seguridad o federar con otros proveedores de identidades, como Active Directory de Microsoft.

AAD es el método preferido para autenticarse en Kusto. Admite varios escenarios de autenticación:
* **Autenticación de usuario** (inicio de sesión interactivo): se usa para autenticar entidades de seguridad humanas.
* **Autenticación de aplicación** (inicio de sesión no interactivo): se usa para autenticar aquellos servicios y aplicaciones que tienen que ejecutarse o autenticarse sin que haya usuarios humanos presentes.

>NOTA: Azure Active Directory no permite la autenticación de cuentas de servicio (que son por definición entidades de AD locales).
El equivalente de AAD de la cuenta de servicio de AD es la aplicación de AAD.

#### <a name="aad-group-principals"></a>Directores del Grupo AAD
Kusto solo admite entidades de seguridad de grupo de seguridad (y no las de grupo de distribución). Si intenta configurar el acceso a una DG en un clúster de Kusto, se producirá un error.

#### <a name="aad-tenants"></a>Inquilinos de AAD


>Si no se especifica explícitamente el inquilino de AAD, Kusto intentará resolverlo desde el `johndoe@fabrikam.com`UPN (UniversalPrincipalName, por ejemplo, ), si se proporciona.
Si la entidad de seguridad no incluye la información del inquilino (no en el formulario UPN), debe mencionarla explícitamente anexando el identificador o el nombre del inquilino al descriptor principal.

**Ejemplos de entidades principales de AAD**

|Inquilino de AAD |Tipo |Sintaxis |
|-----------|-----|-------|
|Implícito (UPN)  |Usuario  |`aaduser=`*UserEmailAddress*
|Explícito (ID)   |Usuario  |`aaduser=`*UserEmailAddress*`;`*TenantId* o `aaduser=` *ObjectID*`;`*TenantId*
|Explícito (Nombre) |Usuario  |`aaduser=`*UserEmailAddress*`;`*TenantName* o `aaduser=` *ObjectID*`;`*TenantName*
|Implícito (UPN)  |Grupo |`aadgroup=`*GroupEmailAddress*
|Explícito (ID)   |Grupo |`aadgroup=`*GroupObjectId*`;`*TenantId* o`aadgroup=`*GroupDisplayName*`;`*TenantId*
|Explícito (Nombre) |Grupo |`aadgroup=`*GroupObjectId*`;`*TenantName* o`aadgroup=`*GroupDisplayName*`;`*TenantName*
|Explícito (UPN)  |Aplicación   |`aadapp`=*ApplicationDisplayName*`;`*TenantId*
|Explícito (Nombre) |Aplicación   |`aadapp=`*ApplicationId*`;`*TenantName*

```kusto
// No need to specify AAD tenant for UPN, as Kusto performs the resolution by itself
.add database Test users ('aaduser=imikeoein@fabrikam.com') 'Test user (AAD)'

// AAD SG on 'fabrikam.com' tenant
.add database Test users ('aadgroup=SGDisplayName;fabrikam.com') 'Test group @fabrikam.com (AAD)'

// AAD App on 'fabrikam.com' tenant - by tenant name
.add database Test users ('aadapp=4c7e82bd-6adb-46c3-b413-fdd44834c69b;fabrikam.com') 'Test app @fabrikam.com (AAD)'
```

### <a name="microsoft-accounts-msas"></a>Cuentas Microsoft (MSA)
Cuentas Microsoft (MSA) es el término que se usa para denominar todas las cuentas de usuario administradas por Microsoft que no pertenecen a ninguna organización `hotmail.com`, `live.com`, `outlook.com`.
Kusto admite la autenticación de usuario en las MSA (tenga en cuenta que no existe el concepto de grupos de seguridad), que se identifican mediante su UPN (nombre de entidad de seguridad universal).
Cuando se configure una entidad de seguridad de MSA en un recurso, Kusto **no** intentará resolver el UPN proporcionado.

**Ejemplos de entidades principales de MSA**

|URL de  |Tipo  |Sintaxis |
|-----|------|-------|
|Live.com |Usuario  |`msauser=`john.doe@live.com`

```kusto
.add database Test users ('msauser=john.doe@live.com') 'Test user (live.com)'
```

