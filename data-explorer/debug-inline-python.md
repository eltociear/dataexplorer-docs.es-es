---
title: Depuración de Python insertado en el lenguaje de consulta de Kusto mediante VS Code - Azure Data Explorer
description: Aprenda a depurar Python insertado en el lenguaje de consulta de Kusto (KQL) mediante VS Code.
author: orspod
ms.author: orspodek
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: conceptual
ms.date: 12/04/2019
ms.openlocfilehash: 808d51ff5bc070449c87d14822fe4f41d2038950
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493492"
---
# <a name="debug-kusto-query-language-inline-python-using-vs-code"></a>Depuración de Python insertado en el lenguaje de consulta de Kusto mediante VS Code

Azure Data Explorer admite la ejecución de código de Python insertado en el lenguaje de consulta de Kusto mediante el [complemento python()](kusto/query/pythonplugin.md). El entorno de ejecución del complemento se hospeda en un espacio aislado, un entorno de Python aislado y seguro. La funcionalidad del complemento python() amplía las funcionalidades nativas del lenguaje de consulta de Kusto con el enorme archivo de paquetes de Python para OSS. Esta extensión permite ejecutar algoritmos avanzados —tales como aprendizaje automático, inteligencia artificial, estadística y serie temporal— como parte de la consulta.

Las herramientas del lenguaje de consulta de Kusto no resultan prácticas para desarrollar y depurar algoritmos de Python. Por lo tanto, puede desarrollar el algoritmo en el entorno de desarrollo integrado de Python que prefiera, como Jupyter, PyCharm, VS o VS Code. Una vez completado el algoritmo, lo copia y lo pega en KQL. Con el fin de mejorar y simplificar este flujo de trabajo, Azure Data Explorer admite la integración entre los clientes Kusto Explorer o la interfaz de usuario web de Kusto y VS Code para la creación y depuración de código Python insertado en KQL. 

> [!NOTE]
> Este flujo de trabajo solo se puede usar para depurar tablas de entrada relativamente pequeñas (unos pocos MB). Por lo tanto, puede que necesite limitar la entrada para la depuración.  Si necesita procesar una tabla grande, limítela para depuración mediante `| take`, `| sample` o `where rand() < 0.x`.

## <a name="prerequisites"></a>Prerrequisitos

1. Instale la [distribución Anaconda](https://www.anaconda.com/distribution/#download-section) de Python. En **Advanced Options** (Opciones avanzadas), seleccione **Add Anaconda to my PATH environment variable** (Agregar Anaconda a mi variable de entorno PATH).
2. Instalar [Visual Studio Code](https://code.visualstudio.com/Download)
3. Instale la [extensión de Python para Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-python.python).

## <a name="run-your-query-in-your-client-application"></a>Ejecución de la consulta en la aplicación cliente

1. En la aplicación cliente, incluya `set query_python_debug;` como prefijo en las consultas que contengan Python insertado.
1. Ejecuta la consulta.
    * Kusto Explorer: VS Code se inicia automáticamente con el script *debug_python.py*.
    * Interfaz de usuario web de Kusto: 
        1. Descargue y guarde *debug_python.py*, *df.txt* y *kargs.txt*. En la ventana, seleccione **Permitir**. **Guarde** los archivos en el directorio seleccionado. 

            ![La interfaz de usuario web descarga los archivos de Python insertados.](media/debug-inline-python/webui-inline-python.png)

        1. Haga clic con el botón derecho en *debug_python.py* y ábralo con VS Code. 
        El script *debug_python.py* contiene el código de Python insertado, procedente de la consulta de KQL, con el prefijo del código de plantilla para inicializar el dataframe de entrada de *df.txt* y el diccionario de parámetros de *kargs.txt*.    
            
1. En VS Code, inicie el depurador de VS Code: **Debug** > **Start Debugging (F5)** (Depurar > Iniciar depuración [F5]) y seleccione la configuración de **Python**. El depurador se iniciará e insertará automáticamente un punto de interrupción para depurar el código insertado.

### <a name="how-does-inline-python-debugging-in-vs-code-work"></a>¿Cómo funciona la depuración de Python insertado en VS Code?

1. La consulta se analiza y se ejecuta en el servidor hasta que se alcanza la cláusula `| evaluate python()` requerida.
1. Se invoca el espacio aislado de Python pero, en lugar de ejecutar el código, se serializa la tabla de entrada, el diccionario de parámetros y el código, y se devuelven al cliente.
1. Estos tres objetos se guardan en tres archivos: *df.txt*, *kargs.txt* y *debug_python.py* en el directorio seleccionado (interfaz de usuario web) o en el directorio % TEMP% del cliente (Kusto Explorer).
1. Se inicia VS Code, ya cargado con el archivo *debug_python.py* que contiene un código de prefijo para inicializar df y kargs desde sus respectivos archivos, seguido del script de Python insertado en la consulta de KQL.

## <a name="query-example"></a>Ejemplo de consulta

1. Ejecute la siguiente consulta de KQL en la aplicación cliente:

    ```kusto
    range x from 1 to 4 step 1
    | evaluate python(typeof(*, x4:int), 
    'exp = kargs["exp"]\n'
    'result = df\n'
    'result["x4"] = df["x"].pow(exp)\n'
    , pack('exp', 4))
    ```

    Vea la tabla resultante:

    | x  | x4  |
    |---------|---------|
    | 1     |   1      |
    | 2     |   16      |
    | 3     |   81      |
    | 4     |    256     |
    
1. Ejecute la misma consulta de KQL en la aplicación cliente con `set query_python_debug;`:

    ```kusto
    set query_python_debug;
    range x from 1 to 4 step 1
    | evaluate python(typeof(*, x4:int), 
    'exp = kargs["exp"]\n'
    'result = df\n'
    'result["x4"] = df["x"].pow(exp)\n'
    , pack('exp', 4))
    ```

1. Se inicia VS Code:

    ![Inicio de VS Code](media/debug-inline-python/launch-vs-code.png)

1. VS Code depura e imprime el dataframe de "result" en la consola de depuración:

    ![Depuración en VS Code](media/debug-inline-python/debug-vs-code.png)

> [!NOTE]
> Puede haber diferencias entre la imagen del espacio aislado de Python y la instalación local. [Compruebe la imagen del espacio aislado de paquetes específicos consultando el complemento](https://github.com/Azure/azure-kusto-analytics-lib/blob/master/Utils/functions/get_modules_version.csl).
