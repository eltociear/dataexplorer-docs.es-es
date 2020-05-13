---
title: 'Espacios aislados: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describen los espacios aislados de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: 6c1f836596c27f0e2901e9f7b109d96aab89cdff
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373516"
---
# <a name="sandboxes"></a>Espacios aislados

El servicio del motor de datos de Kusto es capaz de ejecutar espacios aislados para ejecutar flujos específicos que requieren aislamiento seguro.
Los ejemplos de estos flujos son scripts definidos por el usuario, que se ejecutan con el [complemento de Python](../query/pythonplugin.md) o el [complemento de R](../query/rplugin.md).

Para ejecutar estos espacios aislados, Kusto usa una evolución del proyecto de [Drawbridge](https://www.microsoft.com/research/project/drawbridge/) de Microsoft. Otros servicios de Microsoft usan esta solución para ejecutar objetos definidos por el usuario en un entorno de varios inquilinos.

Los flujos que se ejecutan en espacios aislados no solo están aislados, sino que también son locales (cercanos a los datos), lo que significa que no se agrega ninguna latencia adicional para las llamadas remotas.

## <a name="prerequisites"></a>Prerrequisitos

* El motor de datos **no** debe tener habilitado el [cifrado de discos](../../security.md#data-encryption) .
  * En el futuro se espera la compatibilidad con la ejecución de ambas características en paralelo.
* Los paquetes necesarios (imágenes) para ejecutar los espacios aislados se implementan en cada uno de los nodos del motor de datos y requieren espacio SSD dedicado para ejecutarse.
  * El tamaño estimado es de 20 GB, que, por ejemplo, es aproximadamente 2,5% de la capacidad de SSD de una máquina virtual D14_v2, o 0,7% de la capacidad de SSD de una máquina virtual L16_v1.
  * Esto afecta a la capacidad de datos del clúster y, por tanto, puede afectar al [costo](https://azure.microsoft.com/pricing/details/data-explorer) del clúster.

## <a name="runtime"></a>Tiempo de ejecución

* Un operador de consulta en espacio aislado puede emplear uno o más espacios aislados para su ejecución.
  * Un espacio aislado solo se usa para una sola ejecución, no se comparte entre varias ejecuciones y se elimina una vez completada la ejecución.
  * Los espacios aislados se inicializan de forma diferida en un nodo la primera vez que una consulta requiere un espacio aislado para su ejecución.
    * Esto significa que la primera ejecución de un complemento que usa espacios aislados en un nodo incluirá un breve período de preparación.
  * Cuando se reinicia un nodo (por ejemplo, como parte de una actualización del servicio), se eliminan todos los espacios aislados en ejecución.
* Cada nodo mantiene una cantidad predefinida de espacios aislados que están listos para ejecutar solicitudes entrantes.
  * Una vez que se usa un espacio aislado, se asigna automáticamente uno nuevo para reemplazarlo.
* En caso de que no haya ningún espacio aislado previamente asignado para atender a un operador de consulta, se limitará.
  (consulte [errores](#errors)hasta que se asignen nuevos espacios aislados (puede tardar hasta 10-15 segundos por espacio aislado, en función de la SKU y los recursos disponibles en el nodo de datos).

## <a name="limitations"></a>Limitaciones

Algunas de estas limitaciones se pueden controlar mediante una [Directiva de espacio aislado](../management/sandboxpolicy.md)en el nivel de clúster, para cada tipo de espacio aislado.

* **Número de espacios aislados por nodo:** El número de espacios aislados por nodo es limitado.
  * Las solicitudes que encuentren un estado en el que no haya ningún espacio aislado disponible se limitarán.
* **Red:** Un espacio aislado no puede interactuar con ningún recurso de la máquina virtual o fuera de él.
  * Un espacio aislado no puede interactuar con otro espacio aislado.
* **CPU:** La velocidad máxima de CPU que puede consumir un espacio aislado de los procesadores de su host es limitada (de forma predeterminada `50%` ).
  * Cuando se alcanza este límite, se limita el uso de CPU del espacio aislado, pero continúa la ejecución.
* **Memoria:** La cantidad máxima de memoria RAM que puede consumir un espacio aislado de la RAM del host es limitada (de forma predeterminada `20GB` ).
  * Alcanzar este límite dará como resultado la terminación del espacio aislado (y un error de ejecución de la consulta).
* **Disco:** Un espacio aislado tiene un directorio único adjunto y, además, no puede tener acceso al sistema de archivos del host.
  * Esta carpeta proporciona acceso a la imagen o paquete que coincide con el tipo de espacio aislado (por ejemplo, el paquete de Python o de R que no se puede personalizar).
* **Procesos secundarios:** El espacio aislado está bloqueado para generar procesos secundarios.

> [!NOTE]
> El uso de recursos del espacio aislado no solo depende del tamaño de los datos que se procesan como parte de la solicitud, sino también de la lógica que se ejecuta en el espacio aislado y de la implementación de las bibliotecas que usa.
> (por ejemplo, para `python` los `r` Complementos y, el último significa el script proporcionado por el usuario y las bibliotecas de Python o R que utiliza en tiempo de ejecución).

## <a name="errors"></a>Errores

|Código                      |Message                                                                                        |Posible motivo                                                                                                    |
|--------------------------|-----------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
|E_SB_QUERY_THROTTLED_ERROR|La consulta en espacio aislado se anuló debido a la limitación. Volver a intentar después de algún retroceso puede realizarse correctamente   |No hay ningún espacio aislado disponible en el nodo de destino. Los nuevos espacios aislados deben estar disponibles en unos segundos.         |
|E_SB_QUERY_THROTTLED_ERROR|Los espacios aislados del tipo ' {Kind} ' todavía no se han inicializado                                       |La Directiva de espacio aislado ha cambiado recientemente. Los nuevos espacios aislados que obedecen a la nueva Directiva estarán disponibles en unos segundos.|
