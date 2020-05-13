---
title: 'Complemento de Python: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe el complemento de Python en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 04/01/2020
zone_pivot_group_filename: data-explorer/zone-pivot-groups.json
zone_pivot_groups: kql-flavors
ms.openlocfilehash: d4159af7cd7b45022d29a1c98694dc4ae80451ab
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373129"
---
# <a name="python-plugin"></a>Complemento de Python

::: zone pivot="azuredataexplorer"

El complemento de Python ejecuta una función definida por el usuario (UDF) con un script de Python. El script de Python obtiene los datos tabulares como entrada y se espera que genere una salida tabular.
El tiempo de ejecución del complemento se hospeda en [espacios aislados](../concepts/sandboxes.md)que se ejecutan en los nodos del clúster.

## <a name="syntax"></a>Sintaxis

*T* `|` `evaluate` [ `hint.distribution` `=` ( `single`  |  `per_node` )] `python(` *output_schema* `,` *script* [ `,` *script_parameters*] [ `,` *external_artifacts*]`)`

## <a name="arguments"></a>Argumentos

* *output_schema*: un `type` literal que define el esquema de salida de los datos tabulares, devueltos por el código de Python.
    * El formato es: `typeof(` *columnName* `:` *ColumnType* [,...] `)` , por ejemplo: `typeof(col1:string, col2:long)` .
    * Para extender el esquema de entrada, use la sintaxis siguiente:`typeof(*, col1:string, col2:long)`
* *script*: un `string` literal que es el script de Python válido que se va a ejecutar.
* *script_parameters*: un `dynamic` literal opcional, que es un contenedor de propiedades de pares de nombre y valor que se va a pasar al script de Python como el `kargs` Diccionario reservado (consulte [variables de Python reservadas](#reserved-python-variables)).
* *Hint. Distribution*: sugerencia opcional para que la ejecución del complemento se distribuya en varios nodos del clúster.
  * El valor predeterminado es `single`.
  * `single`: Una instancia única del script se ejecutará en todos los datos de la consulta.
  * `per_node`: Si la consulta antes de que se distribuya el bloque Python, una instancia del script se ejecutará en cada nodo sobre los datos que contiene.
* *external_artifacts*: un `dynamic` literal opcional que es un contenedor de propiedades de pares de nombre & dirección URL para artefactos que son accesibles desde el almacenamiento en la nube y pueden estar disponibles para que el script los use en tiempo de ejecución.
  * Las direcciones URL a las que se hace referencia en esta bolsa de propiedades son necesarias para:
  * Incluirse en la Directiva de [llamada](../management/calloutpolicy.md)del clúster.
    2. Estar en una ubicación disponible públicamente o proporcionar las credenciales necesarias, como se explica en [cadenas de conexión de almacenamiento](../api/connection-strings/storage.md).
  * Los artefactos están disponibles para que el script los consuma desde un directorio temporal local, `.\Temp` , y los nombres proporcionados en el contenedor de propiedades se usan como nombres de archivo locales (vea el [ejemplo](#examples) siguiente).
  * Para obtener más información, consulte el [Apéndice](#appendix-installing-packages-for-the-python-plugin) siguiente.

## <a name="reserved-python-variables"></a>Variables de Python reservadas

Las siguientes variables están reservadas para la interacción entre el lenguaje de consulta de Kusto y el código de Python:

* `df`: Los datos tabulares de entrada (los valores de `T` arriba), como una trama de datos `pandas` .
* `kargs`: El valor del argumento *script_parameters* , como un diccionario de Python.
* `result`: Una `pandas` trama de datos creada por el script de Python cuyo valor se convierte en los datos tabulares que se envían al operador de consulta Kusto que sigue al complemento.

## <a name="onboarding"></a>Incorporación

* El complemento está deshabilitado de forma predeterminada.
* [Aquí](../concepts/sandboxes.md#prerequisites)se enumeran los requisitos previos para habilitar el complemento.
* Habilite o deshabilite el complemento en el [Azure portal en la pestaña **configuración** del clúster](../../language-extensions.md).

## <a name="notes-and-limitations"></a>Notas y limitaciones

* La imagen de espacio aislado de Python se basa en la distribución de *Anaconda 5.2.0* con el motor de *Python 3,6* .
  [Aquí](http://docs.anaconda.com/anaconda/packages/old-pkg-lists/5.2.0/py3.6_win-64/) puede encontrar la lista de sus paquetes (un pequeño porcentaje de paquetes puede ser incompatible con las limitaciones que impone el espacio aislado en el que se ejecuta el complemento).
* La imagen de Python también contiene paquetes de ml comunes: `tensorflow` , `keras` , `torch` , `hdbscan` `xgboost` y otros paquetes útiles.
* De forma predeterminada, el complemento importa *NumPy* (as `np` ) & *pandas* (as `pd` ).  Puede importar otros módulos según sea necesario.
* **[Ingesta de directivas de consulta](../management/data-ingestion/ingest-from-query.md) y [actualización](../management/updatepolicy.md)**
  * Es posible usar el complemento en consultas que son:
      1. Definido como parte de una directiva de actualización, cuya tabla de origen se ingeri al uso de la ingesta *sin transmisión por secuencias* .
      2. Ejecutar como parte de un comando que ingesta de una consulta (por ejemplo, `.set-or-append` ).
  * En los casos anteriores, se recomienda comprobar que el volumen y la frecuencia de la ingesta, así como la complejidad y el uso de los recursos de la lógica de Python están alineados con las [limitaciones de espacio aislado](../concepts/sandboxes.md#limitations)y los recursos disponibles del clúster.
    Si no lo hace, pueden producirse [errores de limitación](../concepts/sandboxes.md#errors).
  * *No* es posible usar el complemento en una consulta que se define como parte de una directiva de actualización, cuya tabla de origen se ingeri mediante la [ingesta de streaming](../../ingest-data-streaming.md).

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

:::image type="content" source="images/plugin/sine-demo.png" alt-text="Demo sinusoidal" border="false":::

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

* Reduzca el conjunto de datos de entrada del complemento a la cantidad mínima necesaria (columnas o filas).
    * Use filtros en el conjunto de datos de origen, cuando sea posible, con el lenguaje de consulta de Kusto.
    * Para realizar un cálculo en un subconjunto de las columnas de origen, solo tiene que proyectar esa columna antes de invocar el complemento.
* Use `hint.distribution = per_node` siempre que la lógica del script sea distribuible.
    * También puede usar el [operador Partition](partitionoperator.md) para particionar el conjunto de datos de entrada.
* Use el lenguaje de consulta de Kusto, siempre que sea posible, para implementar la lógica del script de Python.

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

* Para generar cadenas de varias líneas que contengan el script de Python en `Kusto.Explorer` , copie el script de Python de su editor de Python favorito (*Jupyter*, *Visual Studio Code*, *PyCharm*, etc.). a continuación, puede:
    * Presione *F2* para abrir la ventana **Editar en Python** . Pegue el script en esta ventana. Seleccione **Aceptar**. El script se decorará con comillas y nuevas líneas (por lo que es válido en Kusto) y se pegará automáticamente en la pestaña consulta.
    * Pegue el código de Python directamente en la pestaña consulta, seleccione esas líneas y presione *Ctrl + k*, *Ctrl + S* tecla de acceso rápido para decorarlas como arriba (para invertirla, presione *Ctrl + k*, *Ctrl + M* tecla de acceso rápido). [Esta](../tools/kusto-explorer-shortcuts.md#query-editor) es la lista completa de los métodos abreviados del editor de consultas.
* Para evitar conflictos entre delimitadores de cadena de Kusto y literales de cadena de Python, se recomienda usar caracteres de comillas simples ( `'` ) para literales de cadena de Kusto en consultas de Kusto y caracteres de comillas dobles ( `"` ) para literales de cadena de Python en scripts de Python.
* Use el [operador externaldata](externaldata-operator.md) para obtener el contenido de un script que ha almacenado en una ubicación externa, como Azure BLOB Storage.
  
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

## <a name="appendix-installing-packages-for-the-python-plugin"></a>Apéndice: instalación de paquetes para el complemento de Python

Es posible que tenga que instalar los paquetes por sí solo debido a alguna de las siguientes razones:

* El paquete es privado y es el suyo propio.
* El paquete es público pero no se incluye en la imagen base del complemento.

Puede instalar paquetes siguiendo estos pasos:

1. Requisito previo de una sola vez:
  
  a. Cree un contenedor de blobs para hospedar los paquetes, preferiblemente en la misma región que el clúster.
    * Por ejemplo: `https://artifcatswestus.blob.core.windows.net/python` (suponiendo que el clúster se encuentra en el oeste de EE. UU.)
  
  b. Modifique la Directiva de [llamada](../management/calloutpolicy.md) del clúster para permitir el acceso a esa ubicación.
    * Esto requiere permisos de [AllDatabasesAdmin](../management/access-control/role-based-authorization.md) .
    * Por ejemplo, para habilitar el acceso a un BLOB que `https://artifcatswestus.blob.core.windows.net/python` se encuentra en, el comando que se ejecuta es:

      ```kusto
      .alter-merge cluster policy callout @'[ { "CalloutType": "sandbox_artifacts", "CalloutUriRegex": "artifcatswestus\\.blob\\.core\\.windows\\.net/python/","CanCall": true } ]'
      ```

2. Para paquetes públicos (en [PyPi](https://pypi.org/) u otros canales) a. Descargue el paquete y sus dependencias.
  b. Si es necesario, compile en archivos de rueda ( `*.whl` ):
    * En una ventana de CMD (en el entorno de Python local), ejecute:
      ```python
      pip wheel [-w download-dir] package-name.
      ```

3. Cree un archivo zip que contenga el paquete necesario y sus dependencias:

    * Para paquetes públicos: comprimir los archivos que se descargaron en el paso anterior.
    * Notas:
        * Asegúrese de comprimir los `.whl` archivos y *no* su carpeta principal.
        * Puede omitir `.whl` los archivos de los paquetes que ya existen con la misma versión en la imagen de espacio aislado base.
    * En el caso de los paquetes privados: zip la carpeta del paquete y las de sus dependencias.

4. Cargue el archivo comprimido en un BLOB en la ubicación de artefactos (del paso 1).

5. Llamando al `python` complemento:
    * Especifique el `external_artifacts` parámetro con un contenedor de propiedades de nombre y referencia al archivo zip (la dirección URL del BLOB).
    * En el código de Python en línea: importe `Zipackage` desde `sandbox_utils` y llame a su `install()` método con el nombre del archivo zip.

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

| Identificador | Nombre         |
|----|--------------|
|   1| Tapia de Gary   |
|   2| Emma Evans   |
|   3| Ana Bowen |

---

::: zone-end

::: zone pivot="azuremonitor"

Esta funcionalidad no se admite en Azure Monitor

::: zone-end
