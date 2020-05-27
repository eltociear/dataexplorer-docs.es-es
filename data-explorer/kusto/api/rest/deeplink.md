---
title: 'Vínculos profundos de la interfaz de usuario: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen los vínculos profundos de la interfaz de usuario de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: aa47811bfe8004037cb04e642c234003087617a1
ms.sourcegitcommit: b4d6c615252e7c7d20fafd99c5501cb0e9e2085b
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2020
ms.locfileid: "83863241"
---
# <a name="ui-deep-links"></a>Vínculos profundos de la interfaz de usuario

La API de REST proporciona una funcionalidad de vínculo profundo que permite que `GET` las solicitudes HTTP redirijan al autor de la llamada a una herramienta de interfaz de usuario. Por ejemplo, puede crear un URI que abra la herramienta Kusto. Explorer, lo configura automáticamente para un clúster y una base de datos específicos, ejecuta una consulta específica y muestra los resultados al usuario.

La API de REST de vínculos profundos de la interfaz de usuario:

* Normalmente, el clúster (obligatorio) se define implícitamente, como el servicio que implementa la API de REST, pero se puede invalidar especificando el parámetro de consulta URI `uri` .

* La base de datos (opcional) se especifica como el primer y único fragmento de la ruta de acceso del URI. La base de datos es obligatoria para las consultas y los comandos de control opcionales.

* El comando de consulta o control (opcional) se especifica mediante el parámetro de consulta URI `query` o el parámetro de consulta URI `querysrc` (que señala a un recurso Web que contiene la consulta).
  `query`se puede usar en el texto del propio comando de consulta o control (codificado mediante la codificación de parámetros de consulta HTTP). Como alternativa, se puede usar en la codificación base64 del gzip de la consulta o el texto del comando de control (lo que permite comprimir consultas largas para que se ajusten a los límites de longitud del URI del explorador predeterminado).

* El nombre de la conexión del clúster (opcional) se especifica mediante el parámetro de consulta URI `name` .

* La herramienta de interfaz de usuario se especifica mediante el `web` parámetro de consulta URI opcional.
  `web=0`indica la aplicación de escritorio Kusto. Explorer. `web=1`indica la aplicación web Kusto. webexplorer.
  `web=2`es la versión anterior de Kusto. webexplorer (basada en análisis de Application Insights). `web=3`es Kusto. webexplorer con un perfil vacío (no habrá disponibles pestañas o clústeres abiertos previamente). Por último, el `web` parámetro de consulta se puede reemplazar por `saw=1` para indicar la versión de Kusto. Explorer que se vio.

Estos son algunos ejemplos de vínculos:

* `https://help.kusto.windows.net/`: Cuando un agente de usuario (como un explorador) emite una `GET /` solicitud, se le redirigirá a la herramienta de interfaz de usuario predeterminada configurada para consultar el `help` clúster.
* `https://help.kusto.windows.net/Samples`: Cuando un agente de usuario (como un explorador) emite una `GET /Samples` solicitud, se le redirigirá a la herramienta de interfaz de usuario predeterminada configurada para consultar la `help` base de datos del clúster `Samples` .
* `http://help.kusto.windows.net/Samples?query=StormEvents`: Cuando un usuario (por ejemplo, un explorador) emite una `GET /Samples?query=StormEvents` solicitud, se le redirigirá a la herramienta de interfaz de usuario predeterminada configurada para consultar la `help` base de datos del clúster `Samples` y emitir la `StormEvents` consulta.

> [!NOTE]
> Los URI de vínculo profundo no requieren autenticación, ya que la herramienta de interfaz de usuario, que se usa para la redirección, realiza la autenticación.
> Cualquier `Authorization` encabezado HTTP, si se proporciona, se omite.

> [!IMPORTANT]
> Por motivos de seguridad, las herramientas de interfaz de usuario no ejecutan automáticamente comandos de control, aunque `query` o `querysrc` estén especificados en el vínculo profundo.

## <a name="deep-linking-to-kustoexplorer"></a>Vínculos profundos a Kusto. Explorer

Esta API de REST realiza el redireccionamiento que instala y ejecuta la herramienta de cliente de escritorio Kusto. Explorer con parámetros de inicio especialmente diseñados que abren una conexión a un clúster de Kusto Engine específico y ejecutan una consulta en ese clúster.

* Ruta de acceso: `/` [*nombreDeBaseDeDatos*']
* Solicitud`GET`
* Cadena de consulta:`web=0`

> [!NOTE]
> Vea [vínculos profundos con Kusto. Explorer](../../tools/kusto-explorer-using.md#deep-linking-queries) para obtener una descripción de la sintaxis del URI de redirección para iniciar Kusto. Explorer.

## <a name="deep-linking-to-kustowebexplorer"></a>Vínculos profundos a Kusto. webexplorer

Esta API de REST realiza la redirección a Kusto. webexplorer, una aplicación Web.

* Ruta de acceso: `/` [*nombreDeBaseDeDatos*']
* Solicitud`GET`
* Cadena de consulta:`web=1`

## <a name="specifying-the-query-or-control-command-in-the-uri"></a>Especificar el comando de consulta o control en el URI

Cuando se especifica el parámetro de cadena de consulta de URI `query` , se debe codificar según las reglas HTML de codificación de cadenas de consulta de URI. Como alternativa, el texto del comando de consulta o control se puede comprimir mediante gzip y, a continuación, codificarse mediante la codificación Base64. Esto le permite enviar consultas más largas o comandos de control (ya que el último método de codificación produce URI más cortos).

## <a name="specifying-the-query-or-control-command-by-indirection"></a>Especificar el comando de consulta o control por direccionamiento indirecto

Si el comando de consulta o control es muy largo, incluso codificarlo mediante gzip/Base64 superará la longitud máxima del URI del agente de usuario. Como alternativa, se proporciona el parámetro de cadena de consulta URI `querysrc` y su valor es un URI corto que apunta a un recurso Web que contiene la consulta o el texto del comando de control.

Por ejemplo, puede ser el URI de un archivo hospedado por Azure Blob Storage.

> [!NOTE]
> Si el vínculo profundo es a la herramienta de interfaz de usuario de la aplicación Web, el servicio Web que proporciona el comando de consulta o control (es decir, el servicio que proporciona el `querysrc` URI) debe estar configurado para admitir CORS para `dataexplorer.azure.com` .
>
> Además, si el servicio requiere información de autenticación/autorización, debe proporcionarse como parte del propio URI.
>
> Por ejemplo, si `querysrc` apunta a un BLOB en Azure BLOB Storage, se debe configurar la cuenta de almacenamiento para que admita CORS y hacer que el propio BLOB sea público (por lo que se puede descargar sin notificaciones de seguridad) o agregar una SAS Azure Storage adecuada al URI. La configuración de CORS puede realizarse desde el [Azure portal](https://portal.azure.com/) o desde [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/).
> Consulte [compatibilidad con CORS en Azure Storage](https://docs.microsoft.com/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services).

