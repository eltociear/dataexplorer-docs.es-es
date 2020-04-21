---
title: 'Vínculos profundos de la interfaz de usuario: Explorador de azure Data Explorer ( Azure Data Explorer) Microsoft Docs'
description: En este artículo se describen los vínculos profundos de la interfaz de usuario en El Explorador de datos de Azure.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 10/30/2019
ms.openlocfilehash: c9535ced274ccdbe38f8d9a765e98d11e7b381e5
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81523688"
---
# <a name="ui-deep-links"></a>Enlaces profundos de la interfaz de usuario

La API de REST proporciona una `GET` funcionalidad de vínculo profundo que permite a las solicitudes HTTP redirigir al autor de la llamada a una herramienta de interfaz de usuario. Por ejemplo, puede crear un URI que abra la herramienta Kusto.Explorer, la configure automáticamente para un clúster y una base de datos específicos, ejecute una consulta específica y muestre sus resultados al usuario.

La API REST de vínculos profundos de la interfaz de usuario:

* El clúster (obligatorio) se define normalmente implícitamente, como el servicio que implementa la API de `uri`REST, pero se puede invalidar especificando el parámetro de consulta URI.

* La base de datos (opcional) se especifica como el primer y único fragmento de la ruta de acceso URI. La base de datos es obligatoria para las consultas y opcional para los comandos de control.

* El comando de consulta o control (opcional) se `query`especifica mediante el `querysrc` parámetro de consulta URI o el parámetro de consulta URI (que apunta a un recurso web que contiene la consulta).
  `query`se puede utilizar en el texto del propio comando de consulta o control (codificado mediante la codificación de parámetros de consulta HTTP). Como alternativa, se puede utilizar en la codificación base64 del texto del comando de consulta o control (lo que permite comprimir consultas largas para que se ajusten a los límites de longitud de URI del explorador predeterminados).

* El nombre de la conexión de clúster (opcional) `name`se especifica mediante el parámetro de consulta URI .

* La herramienta de interfaz `web` de usuario se especifica mediante el parámetro de consulta URI opcional.
  `web=0`indica la aplicación de escritorio Kusto.Explorer. `web=1`indica la aplicación web Kusto.WebExplorer.
  `web=2`es la versión anterior de Kusto.WebExplorer (con sede en Application Insights Analytics). `web=3`es el Kusto.WebExplorer con un perfil vacío (no hay pestañas o clústeres previamente abiertos estarán disponibles). Por último, el `web` parámetro de `saw=1` consulta se puede reemplazar por para indicar la versión SAW de Kusto.Explorer.

Aquí hay algunos ejemplos para los links:

* `https://help.kusto.windows.net/`: cuando un agente de usuario (como un explorador) emite una `GET /` solicitud, se `help` le redirigirá a la herramienta de interfaz de usuario predeterminada configurada para consultar el clúster.
* `https://help.kusto.windows.net/Samples`: cuando un agente de usuario (como un explorador) emite una `GET /Samples` solicitud, se `help` le `Samples` redirigirá a la herramienta de interfaz de usuario predeterminada configurada para consultar la base de datos del clúster.
* `http://help.kusto.windows.net/Samples?query=StormEvents`: cuando un usuario (como un `GET /Samples?query=StormEvents` explorador) emite una solicitud, se le redirigirá `Samples` a la `StormEvents` herramienta de interfaz de usuario predeterminada configurada para consultar la base de datos del `help` clúster y emitir la consulta.

> [!NOTE]
> Los URI de vínculo profundo no requieren autenticación, ya que la herramienta de interfaz de usuario que se usa para la redirección realiza la autenticación.
> Cualquier `Authorization` encabezado HTTP, si se proporciona, se omite.

> [!IMPORTANT]
> Por motivos de seguridad, las herramientas de `query` `querysrc` interfaz de usuario no ejecutan automáticamente comandos de control, incluso si se especifican o se especifican en el vínculo profundo.

## <a name="deep-linking-to-kustoexplorer"></a>Enlace profundo a Kusto.Explorer

Esta API de REST realiza la redirección que instala y ejecuta la herramienta de cliente de escritorio Kusto.Explorer con parámetros de inicio especialmente diseñados que abren una conexión a un clúster de motor de Kusto específico y ejecutan una consulta en ese clúster.

* Ruta `/` de acceso: [*DatabaseName*']
* Verbo:`GET`
* Cadena de consulta:`web=0`

> [!NOTE]
> Consulte [Enlace profundo con Kusto.Explorer](../../tools/kusto-explorer.md#deep-linking-queries) para obtener una descripción de la sintaxis de URI de redirección para iniciar Kusto.Explorer.

## <a name="deep-linking-to-kustowebexplorer"></a>Enlace profundo a Kusto.WebExplorer

Esta API de REST realiza la redirección a Kusto.WebExplorer, una aplicación web.

* Ruta `/` de acceso: [*DatabaseName*']
* Verbo:`GET`
* Cadena de consulta:`web=1`

## <a name="specifying-the-query-or-control-command-in-the-uri"></a>Especificar el comando de consulta o control en el URI

Cuando se especifica `query` el parámetro de cadena de consulta URI, debe codificarse según las reglas HTML de codificación de cadena de consulta URI. Como alternativa, el texto del comando query o control se puede comprimir mediante gzip y, a continuación, codificarlo mediante codificación base64. Esto le permite enviar consultas más largas o comandos de control (ya que este último método de codificación da como resultado URI más cortos).

## <a name="specifying-the-query-or-control-command-by-indirection"></a>Especificar el comando de consulta o control por direccionamiento indirecto

Si el comando query o control es muy largo, incluso codificarlo mediante gzip/base64 superará la longitud máxima de URI del agente de usuario. Como alternativa, se proporciona `querysrc` el parámetro de cadena de consulta URI y su valor es un URI corto que apunta a un recurso web que contiene el texto del comando de consulta o control.

Por ejemplo, puede ser el URI de un archivo hospedado por Azure Blob Storage.

> [!NOTE]
> Si el vínculo profundo es a la herramienta de interfaz de usuario de la `querysrc` aplicación web, el servicio web `dataexplorer.azure.com`que proporciona el comando de consulta o control (es decir, el servicio que proporciona el URI) debe configurarse para admitir CORS para .
>
> Además, si ese servicio requiere información de autenticación/autorización, debe proporcionarse como parte del propio URI.
>
> Por ejemplo, `querysrc` si hay puntos en un blob en Azure Blob Storage, se debe configurar la cuenta de almacenamiento para que admita CORS y hacer que el blob en sí sea público (para que se pueda descargar sin notificaciones de seguridad) o agregar una SAS de Azure Storage adecuada al URI. La configuración de CORS se puede realizar desde [Azure Portal](https://portal.azure.com/) o desde el Explorador de [Azure Storage](https://azure.microsoft.com/features/storage-explorer/).
> Consulte [Compatibilidad con CORS en Azure Storage](https://docs.microsoft.com/rest/api/storageservices/cross-origin-resource-sharing--cors--support-for-the-azure-storage-services).

