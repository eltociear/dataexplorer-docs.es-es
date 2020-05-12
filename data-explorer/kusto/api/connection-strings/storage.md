---
title: 'Cadenas de conexión de almacenamiento: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen las cadenas de conexión de almacenamiento en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/23/2020
ms.openlocfilehash: 0456ad7115c51bcdc51b0db82bc9f9b88953be32
ms.sourcegitcommit: 39b04c97e9ff43052cdeb7be7422072d2b21725e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2020
ms.locfileid: "83226217"
---
# <a name="storage-connection-strings"></a>Cadenas de conexión de Storage

Algunos comandos de Kusto indican a Kusto que interactúe con los servicios de almacenamiento externo. Por ejemplo, se puede indicar a Kusto que exporte los datos a un Azure Storage Blob, en cuyo caso es necesario proporcionar los parámetros específicos (como el nombre de la cuenta de almacenamiento o el contenedor de blobs).

Kusto admite los siguientes proveedores de almacenamiento:


* Proveedor de almacenamiento de Azure Storage Blob
* Proveedor de almacenamiento de Azure Data Lake Storage

Cada tipo de proveedor de almacenamiento define un formato de cadena de conexión que se usa para describir los recursos de almacenamiento y cómo obtener acceso a ellos.
Kusto usa un formato de URI para describir estos recursos de almacenamiento y las propiedades necesarias para tener acceso a ellos (por ejemplo, las credenciales de seguridad).


|Proveedor                   |Scheme    |Plantilla de URI                          |
|---------------------------|----------|--------------------------------------|
|Blob de Azure Storage         |`https://`|`https://`*Cuenta* `.blob.core.windows.net/` de *Contenedor*[ `/` *BlobName*] [ `?` *SasKey* \| `;` *AccountKey*]|
|Azure Data Lake Store Gen2|`abfss://`|`abfss://`*Sistema de archivos* `@` *Cuenta* `.dfs.core.windows.net/` de *PathToDirectoryOrFile*[ `;` *CallerCredentials*]|
|Azure Data Lake Store Gen1|`adl://`  |`adl://`*Account*. azuredatalakestore.net/*PathToDirectoryOrFile*[ `;` *CallerCredentials*]|

## <a name="azure-storage-blob"></a>Blob de Azure Storage

Este proveedor es el que se usa con más frecuencia y se admite en todos los escenarios.
Al proveedor se le deben asignar credenciales al acceder al recurso. Existen dos mecanismos compatibles para proporcionar credenciales:

* Proporcione una clave de acceso compartido (SAS) mediante la consulta estándar del Azure Storage Blob ( `?sig=...` ). Use este método cuando Kusto necesite tener acceso al recurso durante un tiempo limitado.
* Proporcione la clave de la cuenta de almacenamiento ( `;ljkAkl...==` ). Use este método cuando Kusto necesite tener acceso al recurso de manera continua.

Ejemplos (tenga en cuenta que esto muestra literales de cadena ofuscados, por lo que no debe exponer la clave de cuenta o SAS):

`h"https://fabrikam.blob.core.windows.net/container/path/to/file.csv;<storage_account_key_text, ends with '=='>"`
`h"https://fabrikam.blob.core.windows.net/container/path/to/file.csv?sv=...&sp=rwd"` 

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

### <a name="azure-data-lake-store-gen-2"></a>Azure Data Lake Store Gen2

Este proveedor admite el acceso a los datos de Azure Data Lake Store Gen 2.

El formato del URI es:

`abfss://`*Sistema de archivos* `@` *StorageAccountName* `.dfs.core.windows.net/` *Ruta de acceso* `;` *CallerCredentials*

Donde:

* _Filesystem_ es el nombre del objeto de sistema de archivos ADLS (que equivale aproximadamente al contenedor de blobs)
* _StorageAccountName_ es el nombre de la cuenta de almacenamiento.
* _Path_ es la ruta de acceso al directorio o archivo al que se está accediendo. el carácter de barra diagonal ( `/` ) se usa como delimitador.
* _CallerCredentials_ indica las credenciales usadas para tener acceso al servicio, tal y como se describe a continuación.

Al obtener acceso a Azure Data Lake Store Gen 2, el autor de la llamada debe proporcionar credenciales válidas para tener acceso al servicio. Se admiten los siguientes métodos para proporcionar credenciales:

* Anexe `;sharedkey=` *ACCOUNTKEY* al URI, donde _AccountKey_ es la clave de la cuenta de almacenamiento.
* Anexe `;impersonate` al URI. Kusto usará la identidad principal del solicitante y lo suplantará para tener acceso al recurso. La entidad de seguridad debe tener las asignaciones de roles de RBAC adecuadas para poder realizar las operaciones de lectura y escritura, tal como se documenta [aquí](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control). (Por ejemplo, el rol mínimo para las operaciones de lectura es el `Storage Blob Data Reader` rol).
* Anexe `;token=` *AADTOKEN* al URI, donde _AadToken_ es un token de acceso de AAD codificado en base 64 (Asegúrese de que el token es para el recurso `https://storage.azure.com/` ).
* Anexe `;prompt` al URI. Kusto solicita las credenciales de usuario cuando necesita tener acceso al recurso. (Preguntar al usuario está deshabilitada para las implementaciones en la nube y solo está habilitada en entornos de prueba).
* Proporcione una clave de acceso compartido (SAS) mediante la consulta estándar de Azure Data Lake Storage Gen 2 ( `?sig=...` ). Use este método cuando Kusto necesite tener acceso al recurso durante un tiempo limitado.



### <a name="azure-data-lake-store-gen-1"></a>Azure Data Lake Store Gen1

Este proveedor admite el acceso a archivos y directorios en Azure Data Lake Store.
Se debe proporcionar con las credenciales (Kusto no usa su propia entidad de seguridad de AAD para tener acceso a Azure Data Lake). Se admiten los siguientes métodos para proporcionar credenciales:

* Anexe `;impersonate` al URI. Kusto usará la identidad principal del solicitante y lo suplantará para tener acceso al recurso.
* Anexe `;token=` *AADTOKEN* al URI, donde *AadToken* es un token de acceso de AAD codificado en base 64 (Asegúrese de que el token es para el recurso `https://management.azure.com/` ).
* Anexe `;prompt` al URI. Kusto solicitará las credenciales de usuario cuando necesite tener acceso al recurso. (Preguntar al usuario está deshabilitada para las implementaciones en la nube y solo está habilitada en entornos de prueba).



