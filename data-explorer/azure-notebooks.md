---
title: Uso de Azure Notebooks para analizar los datos en Azure Data Explorer
description: En este tema se muestra cómo crear una consulta mediante un cuaderno de Azure.
author: orspod
ms.author: orspodek
ms.reviewer: adieldar
ms.service: data-explorer
ms.topic: conceptual
ms.date: 04/01/2020
ms.openlocfilehash: d9564eec41fd78b3506994da2917f1d8765ee266
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373988"
---
# <a name="use-azure-notebooks-to-analyze-data-in-azure-data-explorer"></a>Uso de Azure Notebooks para analizar los datos en Azure Data Explorer

[Azure Notebooks](https://notebooks.azure.com/) es un servicio en la nube de Azure que simplifica la creación y el uso compartido de [cuadernos de Jupyter](https://jupyter.org/). Azure Notebooks facilita la combinación de documentación, código y los resultados de ejecutar este.

> [!Note]
> * En el procedimiento siguiente se usa el cliente de Python en el entorno de Azure Notebooks para consultar Azure Data Explorer. Sin embargo, también puede usar [KQLmagic](kqlmagic.md) para consultar Azure Data Explorer.
> * Algunos usuarios han comunicado problemas de autenticación con Edge; hasta que se resuelvan esos problemas, use un explorador diferente.

## <a name="create-a-project"></a>Crear un proyecto

1. Abra [Azure Notebooks](https://notebooks.azure.com/) en el explorador e inicie sesión.

1. Seleccione la pestaña **My Projects** (Mis proyectos) en el encabezado. 

    [![](media/azurenotebooks/an-myprojects.png "My projects")](media/azurenotebooks/an-myprojects.png#lightbox)

1. Seleccione **+ New project** (+ Nuevo proyecto).
    
1. En el cuadro de diálogo **Create New Project** (Crear nuevo proyecto):
    1. Rellene el campo **Project Name** (Nombre del proyecto).
    1. Desactive la casilla **Public** (Público).
        >[!Important]
        > Si no desactiva esta casilla, el proyecto se expondrá en la red Internet abierta.
    1. Seleccione **Crear**.
    
    ![Creación de un nuevo proyecto](media/azurenotebooks/an-create-new-project-blank.png)

    El identificador del proyecto se crea automáticamente.

## <a name="create-a-notebook"></a>Creación de un cuaderno

1. En el nuevo proyecto, cree un cuaderno. El cuaderno debe usar un [idioma compatible](https://github.com/Azure/azure-kusto-python#minimum-requirements).
Para crear un cuaderno, seleccione **+ New** (+ Nuevo) y elija **Notebook** (Cuaderno).

    ![Creación de un cuaderno](media/azurenotebooks/an-create-new-notebook-menu.png) 

1. En el cuadro de diálogo **Create New Notebook** (Crear un cuaderno), escriba un nombre para el cuaderno.

1. Seleccione **Python 3.6** y elija **New** (Nuevo).
    
    ![Creación de un cuaderno](media/azurenotebooks/an-create-new-notebook.png) 
    
1. En el proyecto, seleccione el nuevo cuaderno.

    ![Selección del nuevo cuaderno](media/azurenotebooks/an-select-notebook.png)

    Espere hasta que se inicialice el kernel de Python. Cuando el icono de círculo relleno cambia a un icono de círculo hueco, puede continuar.

    ![El kernel se inicializa](media/azurenotebooks/an-python-init-icon.png)

1. Para importar el módulo del sistema, escriba los siguientes comandos y seleccione **Run** (Ejecutar):
    ```python
    import sys
    sys.version
    ```

    > [!Note]
    > Para ejecutar una celda, también puede presionar Mayús + Entrar.

1.  Para importar las clases del SDK, escriba los siguientes comandos y seleccione **Run** (Ejecutar):
    ```python
    from azure.kusto.data.request import KustoClient, KustoConnectionStringBuilder
    ```

1.  Para compilar la cadena de conexión e iniciar el cliente de Kusto, escriba los siguientes comandos y seleccione **Run** (Ejecutar):  
    ```python
    kcsb = KustoConnectionStringBuilder.with_aad_device_authentication("https://help.kusto.windows.net")
    kc = KustoClient(kcsb)
    kc.execute("Samples", ".show version")
    ```
1. En el símbolo del sistema, abra una nueva pestaña del explorador para [https://aka.ms/devicelogin](https://aka.ms/devicelogin). 
   
1. Escriba el código de autorización e inicie sesión en su cuenta de AAD (@microsoft.com). Ahora, el cliente de Kusto `kc` puede autenticarse en Azure Data Explorer mediante su identidad.

1. Vuelva al cuaderno para ver el resultado de la autenticación. 

[![](media/azurenotebooks/an-python-commands.png "Python commands")](media/azurenotebooks/an-python-commands.png#lightbox)

## <a name="execute-a-kusto-query"></a>Ejecución de una consulta de Kusto

1. Escriba la consulta de Kusto y seleccione **Run** (Ejecutar). Por ejemplo:

    ```python
    query= "StormEvents | project State, EventType | take 10"
    response = kc.execute("Samples", query)
    for row in response.primary_results[0]:
        print(", ".join(row))
    ```    

[![](media/azurenotebooks/an-commands.png "Python commands")](media/azurenotebooks/an-commands.png#lightbox)

## <a name="next-steps"></a>Pasos siguientes

* [Repositorio Kusto-python de GitHub](https://github.com/Azure/azure-kusto-python)
