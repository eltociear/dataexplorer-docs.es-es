---
title: 'Autenticación de Azure Active Directory (AAD): Explorador de datos de Azure Microsoft Docs'
description: En este artículo se describe la autenticación de Azure Active Directory (AAD) en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 09/13/2019
ms.openlocfilehash: 637252b91af53198b3ee494309857b2d4b6e6828
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522940"
---
# <a name="azure-active-directory-aad-authentication"></a>Autenticación de Azure Active Directory (AAD)

Azure Active Directory (AAD) es el servicio de directorio en la nube multiinquilino preferido de Azure, capaz de autenticar entidades de seguridad o federarse con otros proveedores de identidades, como Microsoft Active Directory.

AAD permite la aplicación de varios tipos (aplicación web, aplicación de escritorio de Windows, aplicaciones universales, aplicaciones móviles, etc.) para autenticar y utilizar uniformemente los servicios de Kusto.

AAD admite una serie de escenarios de autenticación.
Si hay un usuario presente durante la autenticación, se debe autenticar al usuario en AAD mediante la autenticación de usuario de AAD.
En algunos casos, uno quiere un servicio para utilizar Kusto incluso cuando ningún usuario está presente de forma interactiva. En tales casos, se debe autenticar la aplicación mediante el uso de un secreto de aplicación, como se describe en Autenticación de aplicación de AAD.

Kusto admite en general los siguientes métodos de autenticación, incluso a través de sus bibliotecas .NET:

* Autenticación de usuario interactiva: este modo requiere interactividad, ya que, si es necesario, aparecerá la interfaz de usuario de inicio de sesión
* Autenticación de usuario con un token de AAD existente emitido anteriormente para Kusto
* Autenticación de aplicaciones con AppID y secreto compartido
* Autenticación de aplicaciones con certificado X.509v2 instalado localmente o certificado proporcionado en línea
* Autenticación de aplicaciones con un token de AAD existente emitido anteriormente para Kusto
* Autenticación de usuario o aplicación con un token de AAD emitido para otro recurso, siempre que exista confianza entre ese recurso y Kusto

Consulte la referencia de cadenas de [conexión de Kusto](../../api/connection-strings/kusto.md) para obtener instrucciones y ejemplos.

## <a name="user-authentication"></a>Autenticación de usuarios

La autenticación de usuarios se produce cuando el usuario presenta las credenciales a AAD (o a algún proveedor de identidades que federa con AAD, como ADFS) y obtiene un token de seguridad que se puede presentar al servicio Kusto. Al servicio Kusto le da igual cómo se haya obtenido el token de seguridad, lo que le importa es si el token es válido y la información que AAD (o el proveedor de identidades federado) coloca en él.

En el lado del cliente, Kusto admite tanto la autenticación interactiva, en la que la biblioteca cliente de AAD ADAL o un código similar solicitan al usuario que escriba las credenciales. También admite la autenticación basada en token, en la que la aplicación que usa Kusto obtiene un token de usuario válido y lo presenta. Por último, admite un escenario en el que la aplicación que usa Kusto obtiene un token de usuario válido para otro servicio (que no es Kusto), siempre que haya una relación de confianza entre ese recurso y Kusto.

Consulte las [cadenas de conexión de Kusto](../../api/connection-strings/kusto.md) para más información sobre cómo usar las bibliotecas cliente de Kusto y autenticarse con AAD en Kusto.

## <a name="application-authentication"></a>Autenticación de la aplicación

Cuando las solicitudes no están asociadas a ningún usuario concreto o no hay ningún usuario disponible para especificar las credenciales, se puede usar el flujo de autenticación de la aplicación de AAD. En este flujo, la aplicación se autentica en AAD (o en el proveedor de identidades federado) mediante la presentación de cierta información secreta. Los distintos clientes de Kusto admiten los siguientes escenarios:

* Autenticación de la aplicación mediante un certificado X.509v2 instalado localmente.
* Autenticación de la aplicación mediante un certificado X.509v2 proporcionado a la biblioteca cliente como un flujo de bytes.
* Autenticación de la aplicación mediante un identificador de la aplicación de AAD y una clave de la aplicación de AAD (el equivalente de la autenticación con nombre de usuario/contraseña para aplicaciones).
* Autenticación de la aplicación mediante un token de AAD válido obtenido previamente (generado para Kusto).
* Autenticación de la aplicación mediante un token de AAD válido que se ha obtenido previamente y que se ha emitido para otro recurso, siempre que haya una relación de confianza entre ese recurso y Kusto.

## <a name="aad-server-application-permissions"></a>Permisos de aplicación de servidor de AAD

En el caso general, una aplicación de servidor de AAD puede definir varios permisos (por ejemplo, permiso de solo lectura y un permiso de redactor de lectura) y la aplicación cliente de AAD puede decidir qué permisos necesita cuando solicita un token de autorización. Como parte de la adquisición de tokens, se pedirá al usuario que autorice la aplicación cliente de AAD para que actúe en nombre del usuario con autorización para tener estos permisos. Si el usuario lo aprueba, estos permisos se enumerarán en la notificación de ámbito del token que se emite a la aplicación cliente de AAD.



La aplicación cliente de AAD está configurada para solicitar el permiso "Access Kusto" al usuario (que AAD llama "el propietario del recurso").

## <a name="kusto-client-sdk-as-an-aad-client-application"></a>SDK de cliente de Kusto como aplicación cliente de AAD

Cuando las bibliotecas cliente de Kusto invocan a ADAL (la biblioteca cliente de AAD) para adquirir un token para comunicarse con Kusto, proporcionan la siguiente información:

1. El inquilino de AAD, tal como se recibe de la persona que llama
2. El identificador de la aplicación cliente de AAD
3. El identificador de recurso de cliente de AAD
4. El ReplyUrl de AAD (la dirección URL a la que redirigirá el servicio AAD después de que la autenticación se complete correctamente; ADAL entonces captura esta redirección y extrae el código de autorización de él).
5. El URI delhttps://Cluster-and-region.kusto.windows.netclúster (' ').

El token devuelto por ADAL a la biblioteca de cliente de Kusto tiene la aplicación de servidor Kusto AAD como audiencia y el permiso "Acceso a Kusto" como ámbito.

## <a name="authenticating-with-aad-programmatically"></a>Autenticación con AAD mediante programación

En los artículos siguientes se explica cómo autenticarse mediante programación en Kusto con AAD:

* [Cómo aprovisionar una aplicación de AAD](./how-to-provision-aad-app.md)
* [Cómo realizar la autenticación de AAD](./how-to-authenticate-with-aad.md)

