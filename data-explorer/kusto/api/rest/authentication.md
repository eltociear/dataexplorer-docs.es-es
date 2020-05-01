---
title: 'Autenticación a través de HTTPS: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la autenticación a través de HTTPS en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: e0910089d87d6bce6124cb7e4560c2fa7b92b847
ms.sourcegitcommit: 1faf502280ebda268cdfbeec2e8ef3d582dfc23e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/01/2020
ms.locfileid: "82617993"
---
# <a name="authentication-over-https"></a>Autenticación a través de HTTPS

Cuando se usa HTTPS, el servicio admite el encabezado `Authorization` http estándar para realizar la autenticación.

Los métodos de autenticación HTTP admitidos son los siguientes:

* **Azure Active Directory**, a través `bearer` del método.

Al autenticarse mediante Azure AD, el `Authorization` encabezado tiene el formato:

```txt
Authorization: bearer TOKEN
```

Donde `TOKEN` es el token de acceso que el llamador adquiere al comunicarse con el servicio de Azure ad. El token tiene las siguientes propiedades:

* El recurso es el URI de servicio (por ejemplo `https://help.kusto.windows.net`,).
* El extremo de servicio Azure AD es`https://login.microsoftonline.com/TENANT/`

Donde `TENANT` es el identificador o el nombre del inquilino de Azure ad. Por ejemplo, los servicios que se crean en el inquilino de Microsoft `https://login.microsoftonline.com/microsoft.com/`pueden usar. Como alternativa, solo para la autenticación de usuario, la solicitud se puede `https://login.microsoftonline.com/common/`realizar en.

> [!NOTE]
> El punto de conexión de servicio Azure AD cambia cuando se ejecuta en nubes nacionales.
> Para cambiar el punto de conexión, establezca una `AadAuthorityUri` variable de entorno en el URI requerido.

Para obtener más información, consulte la introducción a la [autenticación](../../management/access-control/index.md) y la [Guía para Azure ad la autenticación](../../management/access-control/how-to-authenticate-with-aad.md).
