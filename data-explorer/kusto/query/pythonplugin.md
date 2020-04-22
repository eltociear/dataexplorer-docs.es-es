---
title: Plugin de Python - Explorador de datos de Azure Microsoft Docs
description: En este artículo se describe el complemento Python en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: 5192474779fb712595ff1c25785892bb543fda84
ms.sourcegitcommit: 01eb9aaf1df2ebd5002eb7ea7367a9ef85dc4f5d
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2020
ms.locfileid: "81765711"
---
# <a name="python-plugin"></a>Complemento de Python

::: zone pivot="azuredataexplorer"

El plugin python ejecuta una función definida por el usuario (UDF) mediante una secuencia de comandos de Python. La secuencia de comandos de Python obtiene datos tabulares como entrada y se espera que produzca una salida tabular.
El tiempo de ejecución del complemento se hospeda en [entornos limitados,](../concepts/sandboxes.md)que se ejecutan en los nodos del clúster.

## <a name="syntax"></a>Sintaxis

*T* `|` `=` `per_node` `python(` *output_schema* `,` *script* `,` *script_parameters*`,` *external_artifacts*[`hint.distribution` `single`( | )] output_schema script [ script_parameters ][ external_artifacts ] `evaluate``)`

## <a name="arguments"></a>Argumentos

* *output_schema*: `type` Un literal que define el esquema de salida de los datos tabulares, devueltos por el código Python.
    * El formato `typeof(`es: *ColumnName* `:` *ColumnType* [, ...] `)`, por `typeof(col1:string, col2:long)`ejemplo: .
    * Para extender el esquema de entrada, utilice la sintaxis siguiente:`typeof(*, col1:string, col2:long)`
* *script*: `string` Un literal que es la secuencia de comandos válida de Python que se va a ejecutar.
* *script_parameters*: `dynamic` Un literal opcional, que es una bolsa de propiedades de pares `kargs` nombre/valor que se pasará a la secuencia de comandos de Python como diccionario reservado (consulte [Variables de Python reservadas](#reserved-python-variables)).
* *hint.distribution*: Una sugerencia opcional para que la ejecución del complemento se distribuya entre varios nodos del clúster.
  * El valor predeterminado es `single`.
  * `single`: una única instancia del script se ejecutará en todos los datos de consulta.
  * `per_node`: si se distribuye la consulta antes de que se distribuya el bloque Python, se ejecutará una instancia del script en cada nodo sobre los datos que contiene.
* *external_artifacts*: `dynamic` Un literal opcional que es un contenedor de propiedades de nombre & pares de URL para artefactos que son accesibles desde el almacenamiento en la nube y se puede poner a disposición para que el script lo use en tiempo de ejecución.
  * Las direcciones URL a las que se hace referencia en este contenedor de propiedades son necesarias para:
  * Incluya en la directiva de [llamada](../management/calloutpolicy.md)del clúster.
    2. Estar en una ubicación disponible públicamente, o proporcionar las credenciales necesarias, como se explica en Cadenas de conexión de [almacenamiento](../api/connection-strings/storage.md).
  * Los artefactos están disponibles para que el `.\Temp`script los consuma desde un directorio temporal local y los nombres proporcionados en el contenedor de propiedades se utilizan como nombres de archivo locales (consulte el [ejemplo](#examples) siguiente).
  * Para obtener más información, consulte [el apéndice](#appendix-installing-packages-for-the-python-plugin) a continuación.

## <a name="reserved-python-variables"></a>Variables de Python reservadas

Las siguientes variables están reservadas para la interacción entre el lenguaje de consulta Kusto y el código Python:

* `df`: los datos tabulares `T` de entrada `pandas` (los valores de arriba), como un DataFrame.
* `kargs`: el valor del *script_parameters* argumento, como un diccionario de Python.
* `result`: `pandas` un DataFrame creado por la secuencia de comandos de Python cuyo valor se convierte en los datos tabulares que se envían al operador de consulta Kusto que sigue al complemento.

## <a name="onboarding"></a>Incorporación

* El plugin está deshabilitado de forma predeterminada.
* Los requisitos previos para habilitar el plugin se enumeran [aquí](../concepts/sandboxes.md#prerequisites).
* Habilite o deshabilite el complemento en [Azure Portal en la pestaña **Configuración** del clúster.](https://docs.microsoft.com/azure/data-explorer/language-extensions)

## <a name="notes-and-limitations"></a>Notas y limitaciones

* La imagen de espacio aislado de Python se basa en la distribución *Anaconda 5.2.0* con el motor de *Python 3.6.*
  La lista de sus paquetes se puede encontrar [aquí](http://docs.anaconda.com/anaconda/packages/old-pkg-lists/5.2.0/py3.6_win-64/) (un pequeño porcentaje de paquetes podría ser incompatible con las limitaciones impuestas por el sandbox en el que se ejecuta el complemento).
* La imagen de Python también `tensorflow` `keras`contiene `torch` `hdbscan`paquetes de ML comunes: , , , , `xgboost` y otros paquetes útiles.
* El plugin importa *numpy* (as `np`) `pd`& *pandas* (como ) de forma predeterminada.  Puede importar otros módulos según sea necesario.
* **[Ingestión de](../management/data-ingestion/ingest-from-query.md) [directivas](../management/updatepolicy.md) de consulta y actualización**
  * Es posible utilizar el plugin en las consultas que son:
      1. Definido como parte de una directiva de actualización, cuya tabla de origen se ingsite al uso de la ingesta *que no sea de streaming.*
      2. Ejecutar como parte de un comando que ingesta de `.set-or-append`una consulta (por ejemplo, ).
  * En los dos casos anteriores, se recomienda comprobar que el volumen y la frecuencia de la ingesta, así como la complejidad y la utilización de los recursos de la lógica de Python están alineados con [las limitaciones](../concepts/sandboxes.md#limitations)del espacio aislado y los recursos disponibles del clúster.
    Si no lo hace, puede producirse un error de [limitación.](../concepts/sandboxes.md#errors)
  * *No* es posible utilizar el complemento en una consulta que se define como parte de una política de actualización, cuya tabla de origen se ingesta mediante la ingesta de [streaming.](https://docs.microsoft.com/azure/data-explorer/ingest-data-streaming)

## <a name="examples"></a>Ejemplos

```kusto
range x from 1 to 360 step 1
| evaluate python(
//
typeof(*, fx:double),               //  Output schema: append a new fx column to original table 
//
'result = df\n'                     //  The Python decorated script
'n = df.shape[0]\n'
'g = kargs["gain"]\n'
'f = kargs["cycles"]\n'
'result["fx"] = g * np.sin(df["x"]/n*2*np.pi*f)\n'
//
, pack('gain', 100, 'cycles', 4)    //  dictionary of parameters
)
| render linechart 
```
:::image type="content" source="images/samples/sine-demo.png" alt-text="demostración de seno":::

```kusto
print "This is an example for using 'external_artifacts'"
| evaluate python(
    typeof(File:string, Size:string),
    "import os\n"
    "result = pd.DataFrame(columns=['File','Size'])\n"
    "sizes = []\n"
    "path = '.\\\\Temp'\n"
    "files = os.listdir(path)\n"
    "result['File']=files\n"
    "for file in files:\n"
    "    sizes.append(os.path.getsize(path + '\\\\' + file))\n"
    "result['Size'] = sizes\n"
    "\n",
    external_artifacts = 
        dynamic({"this_is_my_first_file":"https://kustoscriptsamples.blob.core.windows.net/samples/R/sample_script.r",
                 "this_is_a_script":"https://kustoscriptsamples.blob.core.windows.net/samples/python/sample_script.py"})
)
```

| Archivo                  | Size |
|-----------------------|------|
| this_is_a_script      | 120  |
| this_is_my_first_file | 105  |

## <a name="performance-tips"></a>Consejos de rendimiento

* Reduzca el conjunto de datos de entrada del plugin a la cantidad mínima requerida (columnas/filas).
    * Utilice filtros en el conjunto de datos de origen, siempre que sea posible, con el lenguaje de consulta de Kusto.
    * Para realizar un cálculo en un subconjunto de las columnas de origen, proyecte solo las columnas antes de invocar el complemento.
* Utilícelo `hint.distribution = per_node` siempre que la lógica del script sea distribuible.
    * También puede utilizar el operador de [partición](partitionoperator.md) para particionar el conjunto de datos de entrada.
* Utilice el lenguaje de consulta de Kusto, siempre que sea posible, para implementar la lógica de la secuencia de comandos de Python.

    Ejemplo:

    ```kusto    
    .show operations
    | where StartedOn > ago(7d) // Filtering out irrelevant records before invoking the plugin
    | project d_seconds = Duration / 1s // Projecting only a subset of the necessary columns
    | evaluate hint.distribution = per_node python( // Using per_node distribution, as the script's logic allows it
        typeof(*, _2d:double),
        'result = df\n'
        'result["_2d"] = 2 * df["d_seconds"]\n' // Negative example: this logic should have been written using Kusto's query language
      )
    | summarize avg = avg(_2d)
    ```

## <a name="usage-tips"></a>Consejos de uso

* Para generar cadenas de varias líneas `Kusto.Explorer`que contienen la secuencia de comandos de Python en , copie la secuencia de comandos de Python desde su editor de Python favorito (*Jupyter*, *Visual Studio Code*, *PyCharm*, etc.), a continuación:
    * Presione *F2* para abrir la ventana **Editar en Python.** Pegue el script en esta ventana. Seleccione **Aceptar**. El script se decorará con comillas y nuevas líneas (por lo que es válido en Kusto) y se pegará automáticamente en la pestaña de consulta.
    * Pegue el código python directamente en la pestaña de consulta, seleccione esas líneas y presione *Ctrl+K*, *Ctrl+ S* tecla de acceso rápido para decorarlos como arriba (para invertir presione *Ctrl + K*, Ctrl + *M* tecla de acceso rápido). [Aquí](../tools/kusto-explorer-shortcuts.md#query-editor) está la lista completa de accesos directos del Editor de consultas.
* Para evitar conflictos entre delimitadores de cadena de Kusto y literales de cadena de Python, se recomienda usar caracteres de comillas simples (`'`) para literales de cadena de Kusto en consultas de Kusto y caracteres de comillas dobles (`"`) para literales de cadena de Python en scripts de Python.
* Use el [operador externaldata](externaldata-operator.md) para obtener el contenido de un script que ha almacenado en una ubicación externa, como Azure Blob Storage.
  
    **Ejemplo**

    ```kusto
    let script = 
        externaldata(script:string)
        [h'https://kustoscriptsamples.blob.core.windows.net/samples/python/sample_script.py']
        with(format = raw);
    range x from 1 to 360 step 1
    | evaluate python(
        typeof(*, fx:double),
        toscalar(script), 
        pack('gain', 100, 'cycles', 4))
    | render linechart 
    ```

## <a name="appendix-installing-packages-for-the-python-plugin"></a>Apéndice: Instalación de paquetes para el plugin Python

Es posible que deba instalar automáticamente los paquetes debido a cualquiera de las siguientes razones:

* El paquete es privado y es tuyo.
* El paquete es público, pero no está incluido en la imagen base del plugin.

Puede instalar paquetes siguiendo estos pasos:

1. Requisito previo de una sola vez:
  
  a. Cree un contenedor de blobs para hospedar los paquetes, preferiblemente en la misma región que el clúster.
    * Por ejemplo: https://artifcatswestus.blob.core.windows.net/python (suponiendo que el clúster está en el oeste de EE. UU.)
  
  b. Modifique la [directiva de llamada](../management/calloutpolicy.md) del clúster para permitir el acceso a esa ubicación.
    * Esto requiere permisos [AllDatabasesAdmin.](../management/access-control/role-based-authorization.md)
    * Por ejemplo, para habilitar el https://artifcatswestus.blob.core.windows.net/pythonacceso a un blob ubicado en , el comando que se va a ejecutar es:

      ```kusto
      .alter-merge cluster policy callout @'[ { "CalloutType": "sandbox_artifacts", "CalloutUriRegex": "artifcatswestus\\.blob\\.core\\.windows\\.net/python/","CanCall": true } ]'
      ```

2. Para paquetes públicos (en [PyPi](https://pypi.org/) u otros canales) a. Descargue el paquete y sus dependencias.
  b. Si es necesario,`*.whl`compile a los archivos de la rueda ( ):
    * Desde una ventana cmd (en su entorno Python local) ejecute:
      ```python
      pip wheel [-w download-dir] package-name.
      ```

3. Cree un archivo zip que contenga el paquete necesario y sus dependencias:

    * Para paquetes públicos: comprima los archivos que se descargaron en el paso anterior.
    * Notas:
        * Asegúrese de `.whl` comprimir los archivos en sí y *no* su carpeta principal.
        * Puede omitir `.whl` archivos para paquetes que ya existen con la misma versión en la imagen de espacio aislado base.
    * Para paquetes privados: comprima la carpeta del paquete y las de sus dependencias

4. Cargue el archivo comprimido en un blob en la ubicación de artefactos (del paso 1).

5. Llamando `python` al plugin:
    * Especifique `external_artifacts` el parámetro con un contenedor de propiedades de nombre y una referencia al archivo zip (la dirección URL del blob).
    * En el código python `Zipackage` en `sandbox_utils` línea: `install()` importa y llama a su método con el nombre del archivo zip.

### <a name="example"></a>Ejemplo

Instalación del paquete [Faker](https://pypi.org/project/Faker/) que genera datos falsos:

```kusto
range Id from 1 to 3 step 1 
| extend Name=''
| evaluate python(typeof(*),
    'from sandbox_utils import Zipackage\n'
    'Zipackage.install("Faker.zip")\n'
    'from faker import Faker\n'
    'fake = Faker()\n'
    'result = df\n'
    'for i in range(df.shape[0]):\n'
    '    result.loc[i, "Name"] = fake.name()\n',
    external_artifacts=pack('faker.zip', 'https://artifacts.blob.core.windows.net/kusto/Faker.zip?...'))
```

| Identificador | NOMBRE         |
|----|--------------|
|   1| Gary Tapia   |
|   2| Emma Evans   |
|   3| Ashley Bowen |

---

::: zone-end

::: zone pivot="azuremonitor"

Esto no se admite en Azure Monitor

::: zone-end
