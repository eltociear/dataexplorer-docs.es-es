---
title: 'Entidades de seguridad y proveedores de identidades: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las entidades de seguridad y los proveedores de identidades en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/13/2020
ms.openlocfilehash: 4e34c724799cffe38db93869e96fcfae83a92b55
ms.sourcegitcommit: be1bbd62040ef83c08e800215443ffee21cb4219
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2020
ms.locfileid: "84665000"
---
# <a name="principals-and-identity-providers"></a>Entidades de seguridad y proveedores de identidades

El modelo de autorización de Kusto admite varios proveedores de identidades (IDP) y varios tipos de entidad de seguridad.
En este artículo se revisan los tipos de entidad de seguridad admitidos y se muestra su uso con [comandos de asignación de roles](../../management/security-roles.md).

### <a name="azure-active-directory"></a>Azure Active Directory
Azure Active Directory (AAD) es el proveedor de identidades y el servicio de directorio en la nube multiempresa preferido de Azure, capaz de autenticar las entidades de seguridad o de federar con otros proveedores de identidades, como Active Directory de Microsoft.

AAD es el método preferido para autenticarse en Kusto. Admite varios escenarios de autenticación:
* **Autenticación de usuario** (inicio de sesión interactivo): se usa para autenticar entidades de seguridad humanas.
* **Autenticación de aplicación** (inicio de sesión no interactivo): se usa para autenticar aquellos servicios y aplicaciones que tienen que ejecutarse o autenticarse sin que haya usuarios humanos presentes.

> [!NOTE]
> Azure Active Directory no permite la autenticación de cuentas de servicio (que son las entidades de AD locales de definición).
El equivalente de AAD de la cuenta de servicio de AD es la aplicación de AAD.

#### <a name="aad-group-principals"></a>Entidades de seguridad de grupo de AAD
Kusto solo admite entidades de seguridad de grupo de seguridad (y no grupos de distribución). Si intenta configurar el acceso para un grupo de distribución en un clúster de Kusto, se producirá un error.

#### <a name="aad-tenants"></a>Inquilinos de AAD

Si el inquilino de AAD no se especifica explícitamente, Kusto intentará resolverlo desde el UPN (UniversalPrincipalName, por ejemplo, `johndoe@fabrikam.com` ), si se proporciona. Si su entidad de seguridad no incluye la información del inquilino (no en formato UPN), debe mencionarla explícitamente anexando el identificador o el nombre del inquilino al descriptor de la entidad de seguridad.

**Ejemplos de entidades de seguridad de AAD**

|Inquilino de AAD |Tipo |Sintaxis |
|-----------|-----|-------|
|Implícita (UPN)  |Usuario  |`aaduser=`*UserEmailAddress*
|Explícito (ID.)   |Usuario  |`aaduser=`*UserEmailAddress* `;` *Tenantid* o `aaduser=` *objectId* `;` *TenantId*
|Explicit (nombre) |Usuario  |`aaduser=`*UserEmailAddress* `;` *TenantName* o `aaduser=` *objectId* `;` *TenantName*
|Implícita (UPN)  |Grupo |`aadgroup=`*GroupEmailAddress*
|Explícito (ID.)   |Grupo |`aadgroup=`*GroupObjectId* `;` *Tenantid* o `aadgroup=` *GroupDisplayName* `;` *tenantid*
|Explicit (nombre) |Grupo |`aadgroup=`*GroupObjectId* `;` *TenantName* o `aadgroup=` *GroupDisplayName* `;` *TenantName*
|Explícito (UPN)  |Aplicación   |`aadapp`=*ApplicationDisplayName* `;` *TenantId*
|Explicit (nombre) |Aplicación   |`aadapp=`*ApplicationID* `;` *TenantName*

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

**Ejemplos de entidades de seguridad de MSA**

|IdP  |Tipo  |Sintaxis |
|-----|------|-------|
|Live.com |Usuario  |`msauser=`john.doe@live.com`

```kusto
.add database Test users ('msauser=john.doe@live.com') 'Test user (live.com)'
```

