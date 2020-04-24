---
title: Creación de una aplicación de Azure AD en Azure Data Explorer
description: Obtenga información sobre cómo crear una aplicación de Azure AD en Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: herauch
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/01/2020
ms.openlocfilehash: 7bbb8064cd71e61b6fdfb1205e783099ec21c687
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81494260"
---
# <a name="create-an-azure-active-directory-application-registration-in-azure-data-explorer"></a>Creación de un registro de aplicación de Azure Active Directory en Azure Data Explorer

La autenticación de aplicaciones de Azure Active Directory (Azure AD) se utiliza para aplicaciones, como un servicio desatendido o un flujo programado, que necesitan acceder a Azure Data Explorer sin un usuario presente. Si se va a conectar a una base de datos de Azure Data Explorer mediante una aplicación, como una aplicación web, se debe autenticar mediante la autenticación de la entidad de servicio. En este artículo se detalla cómo crear y registrar una entidad de servicio de Azure AD y, a continuación, autorizarla para acceder a una base de datos de Azure Data Explorer.

## <a name="create-azure-ad-application-registration"></a>Creación de un registro de aplicación de Azure AD

La autenticación de la aplicación de Azure AD requiere la creación y el registro de una aplicación en Azure AD. Se crea automáticamente una entidad de servicio cuando se crea el registro de aplicación en un inquilino de Azure AD. 

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y abra la hoja `Azure Active Directory`.

    ![Seleccione Azure Active Directory en el menú del portal.](media/provision-azure-ad-app/create-app-reg-select-azure-active-directory.png)

1. Seleccione la hoja **Registros de aplicaciones** y seleccione **Nuevo registro**.

    ![Inicio de un nuevo registro de aplicaciones](media/provision-azure-ad-app/create-app-reg-new-registration.png)

1. Rellene la información siguiente: 

    * **Nombre** 
    * **Tipos de cuenta admitidos**
    * **URI de redirección** > **Web**
        > [!IMPORTANT] 
        > El tipo de aplicación debe ser **Web**. El identificador URI es opcional y se deja en blanco en este caso.
    * Seleccione **Registrar**.

    ![Botón Registrar en un nuevo registro de aplicaciones](media/provision-azure-ad-app/create-app-reg-register-app.png)

1. Seleccione la hoja **Información general** y copie el valor de **Identificador de aplicación**.

    > [!NOTE]
    > Necesitará el identificador de aplicación para autorizar a la entidad de servicio el acceso a la base de datos.

    ![Copia del identificador de registro de la aplicación](media/provision-azure-ad-app/create-app-reg-copy-applicationid.png)

1. En la hoja **Certificados y secretos**, seleccione **Nuevo secreto de cliente**.

    ![Inicio de la creación del secreto de cliente](media/provision-azure-ad-app/create-app-reg-new-client-secret.png)

    > [!TIP]
    > En este artículo se describe el uso de un secreto de cliente para las credenciales de la aplicación.  También puede usar un certificado X509 para autenticar la aplicación. Seleccione **Cargar certificado** y siga las instrucciones para cargar la parte pública del certificado.

1. Escriba una descripción, un tiempo de expiración y seleccione **Agregar**.

    ![Escritura de los parámetros del secreto de cliente](media/provision-azure-ad-app/create-app-reg-enter-client-secret-details.png)

1. Copie el valor de la clave.

    > [!NOTE]
    > Al salir de esta página, no se podrá acceder al valor de clave.  Necesitará la clave para configurar las credenciales de cliente en la base de datos.

    ![Copia del valor de la clave del secreto de cliente](media/provision-azure-ad-app/create-app-reg-copy-client-secret.png)

Se ha creado la aplicación. Si solo necesita acceder a un recurso autorizado de Azure Data Explorer, como en el ejemplo de programación que aparece a continuación, omita la sección siguiente. Para la compatibilidad con los permisos delegados, consulte [Configuración de permisos delegados para el registro de aplicación](#configure-delegated-permissions-for-the-application-registration).

## <a name="configure-delegated-permissions-for-the-application-registration"></a>Configuración de permisos delegados para el registro de aplicación

Si la aplicación necesita acceder a Azure Data Explorer con las credenciales del usuario que realiza la llamada, configure los permisos delegados para el registro de aplicación. Por ejemplo, si va a crear una API web para acceder a Azure Data Explorer y quiere autenticarse con las credenciales del usuario que *llama a* la API.  

1. En la hoja **Permisos de API**, seleccione **Agregar un permiso**.
1. Seleccione **API usadas en mi organización**. Busque y seleccione **Azure Data Explorer**.

    ![Adición de permisos de API de Azure Data Explorer](media/provision-azure-ad-app/configure-delegated-add-api-permission.png)

1. En **Permisos delegados**, seleccione el cuadro **user_impersonation** y **Agregar permisos.** .

    ![Selección de permisos delegados con suplantación del usuario](media/provision-azure-ad-app/configure-delegated-click-add-permissions.png)     

## <a name="grant-the-service-principal-access-to-an-azure-data-explorer-database"></a>Concesión de acceso a la entidad de servicio a una base de datos de Azure Data Explorer

Ahora que se ha creado el registro de aplicación de la entidad de servicio, debe conceder acceso a la entidad de servicio correspondiente a la base de datos de Azure Data Explorer. 

1. En la [interfaz de usuario web](https://dataexplorer.azure.com/), conéctese a la base de datos y abra una pestaña de consulta.

1. Ejecute el comando siguiente:

    ```kusto
    .add database <DatabaseName> viewers ('<ApplicationId>') '<Notes>'
    ```

    Por ejemplo:
    
    ```kusto
    .add database Logs viewers ('aadapp=f778b387-ba15-437f-a69e-ed9c9225278b') 'Azure Data Explorer App Registration'
    ```

    El último parámetro es una cadena que se muestra como una nota al consultar los roles asociados a una base de datos.
    
    > [!NOTE]
    > Después de crear el registro de aplicación, puede haber un retraso de varios minutos hasta que Azure Data Explorer pueda hacer referencia a él. Si al ejecutar el comando aparece un error que le informa de que no se encuentra la aplicación, espere e inténtelo de nuevo.

Para más información, consulte [administración de roles de seguridad](kusto/management/security-roles.md) y [permisos de ingesta](kusto/api/netfx/kusto-ingest-client-permissions.md).  

## <a name="using-application-credentials-to-access-a-database"></a>Uso de credenciales de aplicación para acceder a una base de datos

Use las credenciales de aplicación para acceder mediante programación a la base de datos mediante la [biblioteca cliente de Azure Data Explorer](kusto/api/netfx/about-kusto-data.md).

```C#
. . .
string applicationClientId = "<myClientID>";
string applicationKey = "<myApplicationKey>";
. . .
var kcsb = new KustoConnectionStringBuilder($"https://{clusterName}.kusto.windows.net/{databaseName}")
    .WithAadApplicationKeyAuthentication(
        applicationClientId,
        applicationKey,
        authority);
var client = KustoClientFactory.CreateCslQueryProvider(kcsb);
var queryResult = client.ExecuteQuery($"{query}");
```

   > [!NOTE]
   > Especifique el identificador de aplicación y la clave del registro de aplicación (entidad de servicio) creada anteriormente.

Para más información, consulte [Autenticación con Azure AD para el acceso a Azure Data Explorer](kusto/management/access-control/how-to-authenticate-with-aad.md) y [Uso de Azure Key Vault con una aplicación web de .NET Core](/azure/key-vault/tutorial-net-create-vault-azure-web-app#create-a-net-core-web-app).

## <a name="troubleshooting"></a>Solución de problemas

### <a name="invalid-resource-error"></a>Error de recurso no válido

Si la aplicación se usa para autenticar a usuarios o aplicaciones para el acceso a Azure Data Explorer, debe configurar permisos delegados para la aplicación de servicio de Azure Data Explorer. Declare que la aplicación puede autenticar a usuarios o aplicaciones para el acceso a Azure Data Explorer. Si no lo hace, se producirá un error similar al siguiente cuando se realice un intento de autenticación:

`AADSTS650057: Invalid resource. The client has requested access to a resource which is not listed in the requested permissions in the client's application registration...`

Deberá seguir las instrucciones descritas en [Configuración de permisos delegados para una aplicación de servicio de Azure Data Explorer](#configure-delegated-permissions-for-the-application-registration).

### <a name="enable-user-consent-error"></a>Error de habilitación del consentimiento del usuario

El administrador del inquilino de Azure AD puede adoptar una directiva que impida a los usuarios del inquilino dar su consentimiento a aplicaciones. Esta situación produce un error similar al siguiente cuando un usuario intenta iniciar sesión en la aplicación:

`AADSTS65001: The user or administrator has not consented to use the application with ID '<App ID>' named 'App Name'`

Deberá ponerse en contacto con el administrador de Azure AD para dar consentimiento a todos los usuarios del inquilino o habilitar el consentimiento del usuario para la aplicación específica.

## <a name="next-steps"></a>Pasos siguientes

* Consulte [Cadenas de conexión de Kusto](kusto/api/connection-strings/kusto.md) para obtener una lista de las cadenas de conexión admitidas.
