---
title: 'Cadenas de conexión de almacenamiento: Explorador de azure Data Explorer . Microsoft Docs'
description: En este artículo se describen las cadenas de conexión de almacenamiento en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/23/2020
ms.openlocfilehash: 909198e42786666d3a26874e18b5cfd57b27ab1f
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81502965"
---
# <a name="storage-connection-strings"></a>Cadenas de conexión de Storage

Algunos comandos de Kusto indican a Kusto que interactúe con los servicios de almacenamiento externo. Por ejemplo, se le puede pedir a Kusto que exporte datos a un blob de Azure Storage, en cuyo caso es necesario proporcionar los parámetros específicos (como el nombre de la cuenta de almacenamiento o el contenedor de blobs).

Kusto es compatible con los siguientes proveedores de almacenamiento:


* Proveedor de almacenamiento de blobs de Azure Storage
* Proveedor de almacenamiento de Azure Data Lake Storage

Cada tipo de proveedor de almacenamiento define un formato de cadena de conexión que se usa para describir los recursos de almacenamiento y cómo tener acceso a ellos.
Kusto usa un formato URI para describir estos recursos de almacenamiento y las propiedades necesarias para tener acceso a ellos (como las credenciales de seguridad).


|Proveedor                   |Scheme    |Plantilla de URI                          |
|---------------------------|----------|--------------------------------------|
|Blob de Azure Storage         |`https://`|`https://`*Contenedor de cuentas*`.blob.core.windows.net/`*Container*[`/`*BlobName*][`?`*SasKey* \| `;` *AccountKey*]|
|Azure Data Lake Store Gen2|`abfss://`|`abfss://`*Cuenta del sistema de*`@`*Account*`.dfs.core.windows.net/`archivos`;`*PathToDirectoryOrFile*[*CallerCredentials*]|
|Azure Data Lake Store Gen1|`adl://`  |`adl://`*Cuenta*.azuredatalakestore.net/*PathToDirectoryOrFile*[`;`*CallerCredentials*]|

## <a name="azure-storage-blob"></a>Blob de Azure Storage

Este proveedor es el más utilizado y se admite en todos los escenarios.
Al tener acceso al recurso, se debe proporcionar al proveedor las credenciales. Existen dos mecanismos admitidos para proporcionar credenciales:

* Proporcione una clave de acceso compartido (SAS) mediante la`?sig=...`consulta estándar del blob de Azure Storage ( ). Utilice este método cuando Kusto necesite acceder al recurso durante un tiempo limitado.
* Proporcione la clave`;ljkAkl...==`de la cuenta de almacenamiento ( ). Utilice este método cuando Kusto necesite acceder al recurso de forma continua.

Ejemplos (tenga en cuenta que esto muestra literales de cadena ofuscados, para no exponer la clave de cuenta o SAS):

`h"https://fabrikam.blob.core.windows.net/container/path/to/file.csv;<storage_account_key_text, ends with '=='>"`
`h"https://fabrikam.blob.core.windows.net/container/path/to/file.csv?sv=...&sp=rwd"` 

## <a name="azure-data-lake-store"></a>Azure Data Lake Store

### <a name="azure-data-lake-store-gen-2"></a>Azure Data Lake Store Gen2

Este proveedor admite el acceso a datos en Azure Data Lake Store Gen 2.

El formato del URI es:

`abfss://`*Filesystem* `@` *StorageAccountName* `.dfs.core.windows.net/` *Path* `;` *CallerCredentials*

Donde:

* _Sistema de archivos_ es el nombre del objeto de sistema de archivos ADLS (aproximadamente equivalente a contenedor de blobs)
* _StorageAccountName_ es el nombre de la cuenta de almacenamiento
* _Ruta_ de acceso es la ruta de`/`acceso al directorio o archivo al que se accede El carácter de barra diagonal ( ) se utiliza como delimitador.
* _CallerCredentials_ indica las credenciales utilizadas para tener acceso al servicio, como se describe a continuación.

Al obtener acceso a Azure Data Lake Store Gen 2, el autor de la llamada debe proporcionar credenciales válidas para obtener acceso al servicio. Se admiten los siguientes métodos de proporcionar credenciales:

* Anexar `;sharedkey=` *AccountKey* al URI, siendo _AccountKey_ la clave de la cuenta de almacenamiento
* Anexar `;impersonate` al URI. Kusto usará la identidad principal del solicitante y la suplantará para acceder al recurso. Principal debe tener las asignaciones de roles RBAC adecuadas para poder realizar las operaciones de lectura y escritura, como se documenta [aquí.](https://docs.microsoft.com/azure/storage/blobs/data-lake-storage-access-control) (Por ejemplo, el rol mínimo `Storage Blob Data Reader` para las operaciones de lectura es el rol).
* Anexe `;token=` *AadToken* al URI, con _AadToken_ como un token de acceso de AAD `https://storage.azure.com/`codificado en base 64 (asegúrese de que el token es para el recurso).
* Anexar `;prompt` al URI. Kusto solicita credenciales de usuario cuando necesita acceder al recurso. (La solicitud del usuario está deshabilitada para las implementaciones en la nube y solo está habilitada en entornos de prueba.)
* Proporcione una clave de acceso compartido (SAS) mediante la consulta`?sig=...`estándar de Azure Data Lake Storage Gen 2 ( ). Utilice este método cuando Kusto necesite acceder al recurso durante un tiempo limitado.



### <a name="azure-data-lake-store-gen-1"></a>Azure Data Lake Store Gen1

Este proveedor admite el acceso a archivos y directorios en Azure Data Lake Store.
Debe proporcionarse con credenciales (Kusto no usa su propia entidad de seguridad de AAD para tener acceso a Azure Data Lake.) Se admiten los siguientes métodos de proporcionar credenciales:

* Anexar `;impersonate` al URI. Kusto utilizará la identidad principal del solicitante y la hará pasar por ella para acceder al recurso
* Anexe `;token=`_AadToken _to el URI, con _AadToken_ como un token de acceso de `https://management.azure.com/`AAD codificado en base 64 (asegúrese de que el token es para el recurso).
* Anexar `;prompt` al URI. Kusto solicitará las credenciales de usuario cuando necesite acceder al recurso. (La solicitud del usuario está deshabilitada para las implementaciones en la nube y solo está habilitada en entornos de prueba.)



