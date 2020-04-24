---
title: 'Autenticación a través de HTTPS: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describe la autenticación a través de HTTPS en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: 4b6fbf5bb34dc3ff52938c7042778a7e49fd5faf
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81503050"
---
# <a name="authentication-over-https"></a>Autenticación a través de HTTPS

Cuando se utiliza HTTPS, `Authorization` el servicio admite el encabezado HTTP estándar para realizar la autenticación.

Los métodos de autenticación HTTP admitidos son:

* **Azure Active Directory** `bearer` , a través del método.

Al autenticar con Azure Active `Authorization` Directory, el encabezado tiene el formato:

```txt
Authorization: bearer TOKEN
```

Dónde `TOKEN` está el token de acceso que adquiere el autor de la llamada comunicándose con el servicio Azure Active Directory, con las siguientes propiedades:

* El recurso es el URI del `https://help.kusto.windows.net`servicio (por ejemplo, ).
* El punto de conexión `https://login.microsoftonline.com/TENANT/`del servicio azure Active Directory es .

Dónde `TENANT` está el identificador o el nombre del inquilino de Azure Active Directory. Por ejemplo, los servicios creados `https://login.microsoftonline.com/microsoft.com/`en el inquilino de Microsoft pueden usar . Como alternativa, solo para la autenticación de `https://login.microsoftonline.com/common/` usuario, la solicitud se puede realizar en su lugar.

> [!NOTE]
> El punto de conexión del servicio Azure Active Directory cambia cuando se ejecuta en nubes nacionales.
> Para cambiar el punto de conexión que `AadAuthorityUri` se usará, establezca una variable de entorno en el URI necesario.

Para obtener más información, consulte la [información general](../../management/access-control/index.md) de autenticación y la guía para la autenticación de Azure [Active Directory.](../../management/access-control/how-to-authenticate-with-aad.md)