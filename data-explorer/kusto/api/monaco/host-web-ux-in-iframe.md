---
title: 'Inserción de la interfaz de usuario Web en un **iframe** : Azure explorador de datos'
description: En este artículo se describe la incrustación de la interfaz de usuario Web en un **iframe** en Azure explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 348a614c8b3336085a59a113f18f6858f024e8c7
ms.sourcegitcommit: e66c5f4b833b4f6269bb7bfa5695519fcb11d9fa
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2020
ms.locfileid: "83630245"
---
# <a name="embed-web-ui-in-an-iframe"></a>Insertar la interfaz de usuario Web en un iframe

La interfaz de usuario Web de Azure Explorador de datos puede incrustarse en un iframe y hospedarse en sitios web de terceros.
 
:::image type="content" source="../images/host-web-ux-in-iframe/web-ux.png" alt-text="Interfaz de usuario Web de Azure Explorador de datos":::

La inserción de Azure Explorador de datos Web UX en su sitio web permite a los usuarios:

- Editar consultas (incluye todas las características del lenguaje, como la coloración e IntelliSense)
- Explorar esquemas de tabla visualmente
- Autenticarse en Azure AD
- Ejecutar consultas
- Mostrar resultados de ejecución de la consulta
- Crear varias pestañas
- Guardar consultas localmente
- Compartir consultas por correo electrónico

Toda la funcionalidad se comprueba para obtener accesibilidad y admite los temas en pantalla oscuros y claros.

## <a name="use-monaco-kusto-or-embed-the-web-ui"></a>¿Usar Mónaco-Kusto o insertar la interfaz de usuario Web?

Mónaco-Kusto ofrece una experiencia de edición como la finalización, la coloración, la refactorización, el cambio de nombre y la definición de un paso a otro. Requiere la creación de una solución para la autenticación, la ejecución de consultas, la visualización de los resultados y la exploración de esquemas, pero ofrece la máxima flexibilidad para la experiencia del usuario que se adapta a sus necesidades.

La inserción de la interfaz de usuario Web de Azure Explorador de datos ofrece una amplia funcionalidad con poco esfuerzo, pero contiene una flexibilidad limitada para la experiencia del usuario. Hay un conjunto fijo de parámetros de consulta que permiten un control limitado sobre la apariencia y el comportamiento del sistema.

## <a name="how-to-embed-the-web-ui-in-an-iframe"></a>Cómo insertar la interfaz de usuario Web en un iframe

### <a name="host-the-website-in-an-iframe"></a>Hospedar el sitio web en un iframe

Agregue el código siguiente al sitio web:

```html
<iframe
  src="https://dataexplorer.azure.com/clusters/<cluster>?ibizaPortal=true"
></iframe>
```

El `ibizaPortal` parámetro de consulta indica a la interfaz de usuario Web de Azure explorador de datos que *no* redirija para obtener un token de autenticación. Esta acción es necesaria, ya que el sitio web de hospedaje es responsable de proporcionar un token de autenticación al iframe incrustado.

Reemplace `<cluster>` por el nombre de host del clúster que desea cargar en el panel de conexión, como `help.kusto.windows.net` . De forma predeterminada, el modo Embedded iframe no proporciona una manera de agregar clústeres desde la interfaz de usuario, ya que se supone que el sitio web de hospedaje es consciente del clúster requerido.

### <a name="handle-authentication"></a>Controlar la autenticación

1. Cuando se establece en ' modo iframe ' ( `ibizaPortal=true` ), la interfaz de usuario Web de Azure explorador de datos no intentará redirigirse para la autenticación. La interfaz de usuario Web usará el mecanismo de publicación de mensajes que utilizan los exploradores para solicitar y recibir un token. Durante la carga de la página, se publicará el siguiente mensaje en la ventana primaria:

   ```javascript
   window.parent.postMessage(
     {
       signature: "queryExplorer",
       type: "getToken"
     },
     "*"
   );
   window.addEventListener(
     "message",
     event => this.handleIncomingMessage(event),
     false
   );
   ```

1. A continuación, escuchará un mensaje con la siguiente estructura:

   ```json
   {
     "type": "postToken",
     "message": "${the actual authentication token}"
   }
   ```

1. El token proporcionado debe ser un [token JWT](https://tools.ietf.org/html/rfc7519) obtenido del [[punto de conexión de autenticación de AAD]](../../management/access-control/how-to-authenticate-with-aad.md#web-client-javascript-authentication-and-authorization).

> [!IMPORTANT]
> La ventana de hospedaje debe actualizar el token antes de la expiración y usar el mismo mecanismo para proporcionar el token actualizado a la aplicación. De lo contrario, una vez que el token expire, se producirá un error en las llamadas de servicio.

### <a name="feature-flags"></a>Marcas de característica

Es posible que la aplicación de hospedaje desee controlar determinados aspectos de la experiencia del usuario. Por ejemplo, oculte el panel conexión o deshabilite la conexión a otros clústeres.
En este escenario, el explorador Web admite marcas de características.

Se puede usar una marca de características en la dirección URL como parámetro de consulta. Si la aplicación de hospedaje desea deshabilitar la adición de otros clústeres, debe usarhttps://dataexplorer.azure.com/?ShowConnectionButtons=false

| establecer                 | Descripción                                                                                                        | Valor predeterminado |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------ | ------------- |
| ShowShareMenu           | Mostrar el elemento de menú compartir                                                                                           | true          |
| ShowConnectionButtons   | Mostrar el botón **Agregar conexión** para agregar un nuevo clúster                                                            | true          |
| ShowOpenNewWindowButton | Mostrar el botón **abrir en** la interfaz de usuario Web, que abre una nueva ventana del explorador y apunta a https://dataexplorer.azure.com con el clúster y la base de datos correctos en el ámbito           | false         |
| ShowFileMenu            | Mostrar el menú Archivo (**descarga**, **pestaña**, **contenido**, etc.)                                                 | true          |
| ShowToS                 | Mostrar **un vínculo a los términos de servicio de Azure explorador de datos** desde el cuadro de diálogo de configuración                             | true          |
| ShowPersona             | Mostrar el nombre de usuario en el menú de configuración, en la esquina superior derecha                                                 | true          |
| IFrameAuth              | Si es true, el explorador Web esperará que el iframe controle la autenticación y proporcione un token a través de un mensaje. Este proceso siempre será true para los escenarios de iframe                                                                                                                                      | false         |
| PersistAfterEachRun     | Normalmente, el explorador Web se conservará en el evento Unload. Cuando se hospeda en iframes, no siempre se activa. Esta marca desencadenará el **estado local de persistencia** después de cada ejecución de la consulta. Como resultado, cualquier pérdida de datos que se produzca, solo afectará al texto que no se haya ejecutado nunca, por lo que se limitará su impacto          | false         |
| ShowSmoothIngestion     | Si es true, Mostrar la experiencia de ingesta de un clic al hacer clic con el botón secundario en una base de datos                                   | true          |
| RefreshConnection       | Si es true, actualiza siempre el esquema al cargar la página y nunca depende del almacenamiento local.                      | false         |
| ShowPageHeader          | Si es true, muestra el encabezado de página que incluye el título y la configuración de Azure Explorador de datos                            | true          |
| HideConnectionPane      | Si es true, el panel de conexión izquierdo no se muestra                                                                  | false         |
| SkipMonacoFocusOnInit   | Corrige el problema de foco al hospedar en iframe                                                                       | false         |

### <a name="feature-flag-presets"></a>Valores preestablecidos de marcas de características

Los valores preestablecidos establecerán un montón de marcas de características a la vez.
Actualmente, solo hay un valor preestablecido.

`IbizaPortal`: Cambia los valores predeterminados de las marcas siguientes.

```json
ShowOpenNewWindowButton: true,
PersistAfterEachRun: true,
IFrameAuth: true,
ShowPageHeader: false,                                 |
```

> [!WARNING]
> Si usa un valor preestablecido, no podrá agregar más marcas de características. Si necesita esa flexibilidad, debe usar marcas de características individuales.
