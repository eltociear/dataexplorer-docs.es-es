---
title: 'Autenticación de Kusto Azure Active Directory (AAD): Explorador de datos de Azure'
description: En este artículo se describe la autenticación Azure Active Directory (AAD) en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
ms.date: 09/13/2019
ms.openlocfilehash: 85d01c9192c71b3274907e5f93e4155b4c98accf
ms.sourcegitcommit: fd3bf300811243fc6ae47a309e24027d50f67d7e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83382223"
---
# <a name="azure-active-directory-aad-authentication"></a>Autenticación Azure Active Directory (AAD)

Azure Active Directory (AAD) es el servicio de directorio en la nube multiinquilino preferido de Azure, capaz de autenticar entidades de seguridad o federarse con otros proveedores de identidades, como Microsoft Active Directory.

AAD permite la aplicación de varios tipos (aplicación Web, aplicación de escritorio de Windows, aplicaciones universales, aplicaciones móviles, etc.) para autenticar y usar servicios de Kusto de forma uniforme.

AAD admite varios escenarios de autenticación.
Si hay un usuario presente durante la autenticación, debe autenticar al usuario en AAD mediante la autenticación de usuario de AAD.
En algunos casos, uno desea que un servicio use Kusto aunque no haya ningún usuario presente de forma interactiva. En tales casos, debe autenticar la aplicación mediante el uso de un secreto de la aplicación, como se describe en autenticación de la aplicación AAD.

Los siguientes métodos de autenticación son compatibles con Kusto en general, incluidos a través de sus bibliotecas de .NET:

* Autenticación de usuario interactivo: este modo requiere interactividad, como si fuera necesario, se abrirá la interfaz de usuario de inicio de sesión
* Autenticación de usuario con un token de AAD existente emitido previamente para Kusto
* Autenticación de aplicaciones con AppID y secreto compartido
* Autenticación de aplicaciones con certificado X. 509v2 instalado localmente o certificado proporcionado en línea
* Autenticación de la aplicación con un token de AAD existente emitido previamente para Kusto
* Autenticación de usuario o aplicación con un token de AAD emitido para otro recurso, siempre que exista confianza entre ese recurso y Kusto

Consulte la referencia de las [cadenas de conexión de Kusto](../../api/connection-strings/kusto.md) para obtener instrucciones y ejemplos.

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

En el caso general, una aplicación de servidor de AAD puede definir varios permisos (por ejemplo, permiso de solo lectura y un permiso de lectura y escritura) y la aplicación cliente de AAD puede decidir qué permisos necesita cuando solicita un token de autorización. Como parte de la adquisición de tokens, se le pedirá al usuario que autorice a la aplicación cliente de AAD que actúe en nombre del usuario con autorización para tener estos permisos. Si el usuario debe aprobar, estos permisos se enumerarán en la demanda de ámbito del token emitido para la aplicación cliente de AAD.



La aplicación cliente de AAD está configurada para solicitar el permiso de acceso Kusto del usuario (que AAD llama "el propietario del recurso").

## <a name="kusto-client-sdk-as-an-aad-client-application"></a>SDK de cliente de Kusto como aplicación cliente de AAD

Cuando las bibliotecas cliente de Kusto invocan a ADAL (la biblioteca cliente de AAD) para adquirir un token para comunicarse con Kusto, proporcionan la siguiente información:

1. El inquilino de AAD, tal como se recibió del autor de la llamada
2. El identificador de la aplicación cliente de AAD
3. El identificador de recurso de cliente de AAD
4. ReplyUrl de AAD (la dirección URL a la que se redirigirá el servicio AAD después de que la autenticación se complete correctamente; A continuación, ADAL captura este redireccionamiento y extrae el código de autorización de él.
5. El URI del clúster (' https://Cluster-and-region.kusto.windows.net ').

El token devuelto por ADAL a la biblioteca de cliente de Kusto tiene la aplicación de servidor AAD de Kusto como audiencia y el permiso "Access Kusto" como ámbito.

## <a name="authenticating-with-aad-programmatically"></a>Autenticación con AAD mediante programación

En los siguientes artículos se explica cómo autenticarse en Kusto con AAD mediante programación:

* [Aprovisionamiento de una aplicación de AAD](./how-to-provision-aad-app.md)
* [Cómo realizar la autenticación de AAD](./how-to-authenticate-with-aad.md)
