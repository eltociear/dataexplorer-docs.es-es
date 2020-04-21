---
title: Incrustar la interfaz de usuario web en un iframe - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe Incrustar interfaz de usuario web en un iframe en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 02/19/2020
ms.openlocfilehash: 9c5469a577c3e09cd433ec250f9215e5f450d5f5
ms.sourcegitcommit: 653bfb3edf32553c52ef36b339c8b80713a601b0
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2020
ms.locfileid: "81524470"
---
# <a name="embed-web-ui-in-an-iframe"></a>Incrustar interfaz de usuario web en un iframe

La interfaz de usuario web de Azure Data Explorer se puede incrustar en un iframe y hospedarse en sitios web de terceros.

![texto alternativo](../images/web-ux.jpg "Interfaz de usuario web de Azure Data Explorer")

La inserción de la experiencia de usuario web de Azure Data Explorer en el sitio web permite a los usuarios hacer lo siguiente:

- Editar consultas (incluye todas las características del lenguaje, como la coloración y el intellisense)
- Explore visualmente los esquemas de tabla
- Autenticar a AAD
- Ejecutar consultas
- Mostrar los resultados de la ejecución de consultas
- Crear varias pestañas
- Guardar consultas localmente
- Compartir consultas por correo electrónico

Todas las funciones se prueban para la accesibilidad y es compatible con el tema oscuro y claro.

## <a name="use-monaco-kusto-or-embed-the-web-ui"></a>¿Utiliza Monaco-Kusto o incrusta la interfaz de usuario web?

Monaco-Kusto le ofrece una experiencia de edición como finalización, coloración, refactorización, cambio de nombre y definición, requiere que cree una solución para la autenticación, la ejecución de consultas, la visualización de resultados y la exploración de esquemas a su alrededor. Por otro lado, obtiene total flexibilidad para crear la experiencia de usuario que se adapte a sus necesidades.

La inserción de la interfaz de usuario web de Azure Data Explorer ofrece una amplia funcionalidad con poco esfuerzo, pero contiene una flexibilidad limitada con respecto a la experiencia del usuario. Hay un conjunto fijo de parámetros de consulta que permiten un control limitado sobre la apariencia y el comportamiento del sistema.

## <a name="how-to-embed-the-web-ui-in-an-iframe"></a>Cómo incrustar la interfaz de usuario web en un iframe

### <a name="host-the-website-in-iframe"></a>Alojar el sitio web en iframe

Agregue el siguiente código a su sitio web:

```html
<iframe
  src="https://dataexplorer.azure.com/clusters/<cluster>?ibizaPortal=true"
></iframe>
```

El `ibizaPortal` parámetro de consulta indica a la interfaz de usuario web de Azure Data Explorer que _no_ redirija para obtener un token de autenticación. Esto es necesario ya que el sitio web de alojamiento es responsable de proporcionar un token de autenticación al iframe incrustado.

Reemplace `<cluster>` por el nombre de host del clúster que `example: help.kusto.windows.net`desea cargar en el panel de conexión (para ). De forma predeterminada, el modo incrustado en iframe no proporciona una manera de agregar clústeres desde la interfaz de usuario, ya que la suposición es que el sitio web de hospedaje es consciente del clúster necesario.

### <a name="handle-authentication"></a>Controlar la autenticación

1. Cuando se establece en 'modo iframe' (`ibizaPortal=true`), la interfaz de usuario web de Azure Data Explorer no intentará redirigir la autenticación. La interfaz de usuario web usará el mecanismo de publicación de mensajes que los exploradores usan para solicitar y recibir un token. Durante la carga de la página, se publicará el siguiente mensaje en la ventana principal:

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

1. El token proporcionado debe ser un [token JWT](https://tools.ietf.org/html/rfc7519) obtenido del [punto de conexión de [autenticación de AAD].](../../management/access-control/how-to-authenticate-with-aad.md#web-client-javascript-authentication-and-authorization)

> [!IMPORTANT]
> Es responsabilidad de la ventana de hospedaje actualizar el token antes de la expiración y usar el mismo mecanismo para proporcionar el token actualizado a la aplicación. De lo contrario, una vez que expire el token, las llamadas de servicio siempre producirán un error.

### <a name="feature-flags"></a>Banderas de características

Es posible que la aplicación de hospedaje desee contorl ciertos aspectos de la experiencia del usuario. Por ejemplo: oculte el panel de conexión o deshabilite la conexión a otro clúster.
Para este escenario, el explorador web admite marcas de características.

Un indicador de entidad se puede utilizar en la url como parámetro de consulta. Por ejemplo, si la aplicación de hospedaje desea deshabilitar la adición de otro clúster, debe usarhttps://dataexplorer.azure.com/?ShowConnectionButtons=false

| establecer                 | Descripción                                                                                                                                                                                                                                                                                       | Valor predeterminado |
| ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------- |
| ShowShareMenu           | Mostrar el elemento del menú compartir                                                                                                                                                                                                                                                                          | true          |
| ShowConnectionButtons   | Mostrar el botón Agregar conexión para agregar un nuevo clúster                                                                                                                                                                                                                                               | true          |
| ShowOpenNewWindowButton | Mostrar "abrir en el botón de la interfaz de usuario web. esto abrirá una nueva ventana del https://dataexplorer.azure.com navegador que apuntará con el clúster y la base de datos correctos en el ámbito                                                                                                                                   | False         |
| ShowFileMenu            | Mostrar el menú de archivos (descargar contenido de la pestaña ETC)                                                                                                                                                                                                                                                     | true          |
| ShowToS                 | Mostrar vínculo a los términos de servicio para el explorador de datos azure desde el cuadro de diálogo de configuración                                                                                                                                                                                                                | true          |
| ShowPersona             | mostrar el nombre de usuario en la parte superior derecha y desde el menú de configuración                                                                                                                                                                                                                                        | true          |
| IFrameAuth              | Si es true, el explorador web esperará que el iframe controle la autenticación y proporcione un token a través del mensaje. Esto siempre será cierto para escenarios de iframe                                                                                                                                              | False         |
| PersistAfterEachRun     | Por lo general, el explorador web persistirá en el evento onunload. al alojar en iframes, no se activa en algunos casos. por lo tanto, esta marca desencadenará el estado local persituante después de cada ejecución de consulta. Por lo tanto, cualquier pérdida de datos que se produzca solo afectará al texto que nunca se había ejecutado, lo que limitará el impacto. | False         |
| ShowSmoothIngestion     | Si es true, muestre la experiencia de ingesta de 1 clic al hacer clic con el botón derecho en una base de datos                                                                                                                                                                                                                  | true          |
| RefreshConnection       | Si es true, siempre actualiza el esquema al cargar la página y nunca depende del almacenamiento local                                                                                                                                                                                                      | False         |
| ShowPageHeader          | Si es true, muestra el encabezado de página (que incluye el título, la configuración y otros) del Explorador de datos de Azure)                                                                                                                                                                                                 | true          |
| HideConnectionPane      | Si es true, no muestra el panel de conexión izquierdo                                                                                                                                                                                                                                                | False         |
| SkipMonacoFocusOnInit   | Corrige el problema de enfoque al alojar en iframe                                                                                                                                                                                                                                                          | False         |

### <a name="feature-flag-presets"></a>Preajustes de la bandera de la característica

ajustes preestablecidos establecerán un montón de indicadores de características a la vez.
Hoy tenemos un solo preset.

`IbizaPortal`- Cambia los siguientes indicadores de los valores predeterminados:

```json
ShowOpenNewWindowButton: true,
PersistAfterEachRun: true,
IFrameAuth: true,
ShowPageHeader: false,                                 |
```

> [!WARNING]
> Si utilizas un ajuste preestablecido, no podrás añadir marcas de características adicionales encima de él. Si necesita esa flexibilidad, debe utilizar indicadores de entidades individuales.
