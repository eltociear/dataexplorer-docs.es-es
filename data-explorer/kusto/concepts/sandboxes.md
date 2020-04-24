---
title: Espacios aislados - Explorador de azure Data Explorer ? Microsoft Docs
description: En este artículo se describen los espacios aislados en Azure Data Explorer.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: faa7a8225aecd5992e3064fd10cfcc0e559c5127
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81522974"
---
# <a name="sandboxes"></a>Espacios aislados

El servicio Motor de datos de Kusto es capaz de ejecutar entornos limitados para ejecutar flujos específicos que requieren aislamiento seguro.
Ejemplos de estos flujos son scripts definidos por el usuario, que se ejecutan utilizando el [plugin Python](../query/pythonplugin.md) o el [complemento R.](../query/rplugin.md)

Para ejecutar estos entornos sandbox, Kusto utiliza una evolución del proyecto [Drawbridge](https://www.microsoft.com/research/project/drawbridge/) de Microsoft. Otros servicios de Microsoft usan esta solución para ejecutar objetos definidos por el usuario en un entorno multiinquilino.

Los flujos que se ejecutan en entornos limitados no solo están aislados, sino que también son locales (cerca de los datos), lo que significa que no hay latencia adicional agregada para las llamadas remotas.

## <a name="prerequisites"></a>Prerrequisitos

* El motor de datos **no** debe tener habilitado el cifrado de [disco.](https://docs.microsoft.com/azure/data-explorer/security#data-encryption)
  * Se espera compatibilidad con ambas características que se ejecutan en paralelo en el futuro.
* Los paquetes necesarios (imágenes) para ejecutar los entornos limitados se implementan en cada uno de los nodos del motor de datos y requieren espacio SSD dedicado para ejecutarse
  * El tamaño estimado es de 20 GB, lo que, por ejemplo, es aproximadamente el 2,5% de la capacidad de SSD de una máquina virtual D14_v2, o el 0,7% de la capacidad de SSD de una máquina virtual L16_v1.
  * Esto afecta a la capacidad de datos del clúster y, por lo tanto, puede afectar al [costo](https://azure.microsoft.com/pricing/details/data-explorer) del clúster.

## <a name="runtime"></a>Tiempo de ejecución

* Un operador de consulta de espacio aislado puede utilizar uno o varios entornos limitados para su ejecución.
  * Solo se usan para una sola ejecución, no se comparten entre varias ejecuciones y se eliminan una vez que se completa la ejecución.
* Cada nodo mantiene una cantidad predefinida de espacios aislados que están listos para ejecutar solicitudes entrantes.
  * Una vez que se utiliza un entorno limitado, se asigna automáticamente uno nuevo para reemplazarlo.
* En caso de que no haya espacios aislados asignados previamente disponibles para servir a un operador de consulta, se limitará.
  (consulte [Errores](#errors), hasta que se asignen nuevos entornos limitados (podría tardar entre 10 y 15 segundos por espacio aislado, según la SKU y los recursos disponibles en el nodo de datos).

## <a name="limitations"></a>Limitaciones

Algunas de estas limitaciones se pueden controlar mediante una directiva de [espacio aislado](../management/sandboxpolicy.md)de nivel de clúster para cada tipo de espacio aislado.

* **Número de espacios aislados por nodo:** El número de espacios aislados por nodo es limitado.
  * Las solicitudes que encuentran un estado en el que no hay ningún entorno limitado disponible se limitarán.
* **Red:** Un entorno limitado no puede interactuar con ningún recurso de la máquina virtual o fuera de ella.
  * Un entorno limitado no puede interactuar con otro entorno limitado.
* **CPU:** La velocidad máxima de CPU que un entorno limitado puede consumir `50%`de los procesadores de su host es limitada (de forma predeterminada, ).
  * Cuando se alcanza este límite, el uso de CPU del entorno limitado se limita, pero la ejecución continúa.
* **Memoria:** La cantidad máxima de RAM que un entorno limitado puede consumir `20GB`de la RAM de su host es limitada (de forma predeterminada a ).
  * Alcanzar este límite resultará con la terminación del espacio aislado (y un error de ejecución de consulta).
* **Disco:** Un entorno limitado tiene un directorio único asociado, además, no puede acceder al sistema de archivos del host.
  * Esta carpeta proporciona acceso a la imagen/paquete que coincide con el tipo de sandbox (por ejemplo, el paquete de Python o R no personalizable).
* **Procesos infantiles:** El entorno limitado está bloqueado para no generar procesos secundarios.

> [!NOTE]
> La utilización de recursos de espacio aislado depende no solo del tamaño de los datos que se procesan como parte de la solicitud, sino también de la lógica que se ejecuta en el entorno limitado y de la implementación de las bibliotecas que utiliza.
> (por ejemplo, para `python` `r` los plugins y, este último, significa el script proporcionado por el usuario y las bibliotecas de Python o R que consume en tiempo de ejecución)

## <a name="errors"></a>Errors

|Código                      |Message                                                                                        |Posible razón                                                                                                    |
|--------------------------|-----------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
|E_SB_QUERY_THROTTLED_ERROR|La consulta de espacio aislado se anuló debido a la limitación. Reintentar después de algún retroceso podría tener éxito   |No hay espacios aislados disponibles en el nodo de destino. Los nuevos sandboxes deberían estar disponibles en pocos segundos         |
|E_SB_QUERY_THROTTLED_ERROR|Aún no se han inicializado los espacios aislados de tipo ''kind''                                       |La directiva de espacio aislado ha cambiado recientemente. Los nuevos sandboxes que obedezcan la nueva política estarán disponibles en pocos segundos|