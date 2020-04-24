---
title: Consulta de datos mediante la biblioteca de Python de Azure Data Explorer
description: En este artículo aprenderá a utilizar Python para consultar datos de Azure Data Explorer.
author: orspod
ms.author: orspodek
ms.reviewer: mblythe
ms.service: data-explorer
ms.topic: conceptual
ms.date: 08/05/2019
ms.openlocfilehash: ebd65f2dcbb0040b764290627bbfd2901aa9a7d3
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81494212"
---
# <a name="query-data-using-the-azure-data-explorer-python-library"></a>Consulta de datos mediante la biblioteca de Python de Azure Data Explorer

En este artículo, se van a consultar datos mediante Azure Data Explorer. El Explorador de datos de Azure es un servicio de exploración de datos altamente escalable y rápido para datos de telemetría y registro.

El Explorador de datos de Azure proporciona un [biblioteca de cliente de datos para Python](https://github.com/Azure/azure-kusto-python/tree/master/azure-kusto-data). Esta biblioteca permite consultar los datos desde el código. Conéctese a una tabla en el *clúster de ayuda* que se ha configurado para facilitar el aprendizaje. Puede consultar una tabla de ese clúster y devolver los resultados.

Este artículo también está disponible como [Azure Notebook](https://notebooks.azure.com/ManojRaheja/libraries/KustoPythonSamples/html/QueryKusto.ipynb).

## <a name="prerequisites"></a>Prerequisites

* [Python 3.4+](https://www.python.org/downloads/)

* Una cuenta de correo electrónico organizativa que sea miembro de Azure Active Directory (AAD)

## <a name="install-the-data-library"></a>Instalar la biblioteca de datos

Instale *azure-kusto-data*.

```
pip install azure-kusto-data
```

## <a name="add-import-statements-and-constants"></a>Incorporación de instrucciones de importación y constantes

Importe clases de la biblioteca, así como *Pandas*, una biblioteca de análisis de datos.

```python
from azure.kusto.data.request import KustoClient, KustoConnectionStringBuilder
from azure.kusto.data.exceptions import KustoServiceError
from azure.kusto.data.helpers import dataframe_from_result_table
import pandas as pd
```

Para autenticar una aplicación, el Explorador de datos de Azure usa el identificador del inquilino de AAD. Para buscar el identificador de inquilino, use la dirección URL siguiente, sustituyendo su dominio por *SuDominio*.

```
https://login.windows.net/<YourDomain>/.well-known/openid-configuration/
```

Por ejemplo, si el nombre de dominio es *contoso.com*, la dirección URL es: [https://login.windows.net/contoso.com/.well-known/openid-configuration/](https://login.windows.net/contoso.com/.well-known/openid-configuration/). Haga clic en esta dirección URL para ver los resultados. la primera línea es como sigue.

```
"authorization_endpoint":"https://login.windows.net/6babcaad-604b-40ac-a9d7-9fd97c0b779f/oauth2/authorize"
```

En este caso es el id. de inquilino es `6babcaad-604b-40ac-a9d7-9fd97c0b779f`. Establezca el valor de AAD_TENANT_ID antes de ejecutar este código.

```python
AAD_TENANT_ID = "<TenantId>"
KUSTO_CLUSTER = "https://help.kusto.windows.net/"
KUSTO_DATABASE = "Samples"
```

Ahora, cree la cadena de conexión. En este ejemplo se utiliza la autenticación de dispositivos para acceder al clúster. También puede usar el [certificado de la aplicación de AAD](https://github.com/Azure/azure-kusto-python/blob/master/azure-kusto-data/tests/sample.py#L24), la [clave de aplicación de AAD](https://github.com/Azure/azure-kusto-python/blob/master/azure-kusto-data/tests/sample.py#L20) y el [usuario y la contraseña de AAD](https://github.com/Azure/azure-kusto-python/blob/master/azure-kusto-data/tests/sample.py#L34).

```python
KCSB = KustoConnectionStringBuilder.with_aad_device_authentication(
    KUSTO_CLUSTER)
KCSB.authority_id = AAD_TENANT_ID
```

## <a name="connect-to-azure-data-explorer-and-execute-a-query"></a>Conectarse al Explorador de datos de Azure y ejecutar una consulta

Ejecute una consulta en el clúster y almacene el resultado en un marco de datos. Cuando se ejecuta este código, devuelve un mensaje similar al siguiente: *Para iniciar sesión, use un explorador web para abrir la página https://microsoft.com/devicelogin y escriba el código F3W4VWZDM para autenticarse*. Siga los pasos para iniciar sesión y después vuelva a ejecutar el siguiente bloque de código.

```python
KUSTO_CLIENT = KustoClient(KCSB)
KUSTO_QUERY = "StormEvents | sort by StartTime desc | take 10"

RESPONSE = KUSTO_CLIENT.execute(KUSTO_DATABASE, KUSTO_QUERY)
```

## <a name="explore-data-in-dataframe"></a>Exploración de los datos en DataFrame

Después de especificar un inicio de sesión, la consulta devuelve resultados, que se almacenan en un marco de datos. Puede trabajar con los resultados como hace con cualquier otro marco de datos.

```python
df = dataframe_from_result_table(RESPONSE.primary_results[0])
df
```

Debe ver los diez primeros resultados de la tabla StormEvents.

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Ingesta de datos mediante la biblioteca de Python de Azure Data Explorer](python-ingest-data.md)
