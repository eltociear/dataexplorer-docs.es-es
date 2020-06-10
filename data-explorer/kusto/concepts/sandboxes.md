---
title: 'Espacios aislados: Azure Explorador de datos'
description: En este artículo se describen los espacios aislados de Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.date: 03/30/2020
ms.openlocfilehash: 7dbbd4b94169e6b1e23547c7a6b9ed94cf5e70ca
ms.sourcegitcommit: be1bbd62040ef83c08e800215443ffee21cb4219
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2020
ms.locfileid: "84664971"
---
# <a name="sandboxes"></a>Espacios aislados

El servicio de motor de datos de Kusto puede ejecutar espacios aislados para flujos específicos que requieren aislamiento seguro.
Ejemplos de estos flujos son scripts definidos por el usuario que se ejecutan con el [complemento de Python](../query/pythonplugin.md) o el [complemento de R](../query/rplugin.md).

Para ejecutar estos espacios aislados, Kusto usa una versión evolucionda del proyecto [Drawbridge](https://www.microsoft.com/research/project/drawbridge/) de Microsoft. Otros servicios de Microsoft usan esta solución para ejecutar objetos definidos por el usuario en un entorno de varios inquilinos.

Los flujos que se ejecutan en espacios aislados no están aislados. También son locales (cerca de los datos). Esto significa que no hay ninguna latencia adicional agregada para las llamadas remotas.

## <a name="prerequisites"></a>Requisitos previos

* El motor de datos no debe tener habilitado el [cifrado de discos](../../security.md#data-encryption) .
  * En el futuro se espera la compatibilidad con ambas características que se ejecutan en paralelo.
* Los paquetes necesarios (imágenes) para ejecutar los espacios aislados se implementan en cada uno de los nodos del motor de datos y requieren un espacio SSD dedicado para ejecutarse.
  * El tamaño estimado es de 20 GB, que es aproximadamente 2,5% de la capacidad de SSD de una máquina virtual D14_v2, por ejemplo, o 0,7% de la capacidad de SSD de una máquina virtual L16_v1.
  * Esto afecta a la capacidad de datos del clúster y puede afectar al [costo](https://azure.microsoft.com/pricing/details/data-explorer) del clúster.

## <a name="runtime"></a>Tiempo de ejecución

* Un operador de consulta en espacio aislado puede usar uno o varios espacios aislados para su ejecución.
  * Un espacio aislado solo se usa para una sola ejecución, no se comparte entre varias ejecuciones y se elimina una vez completada la ejecución.
  * Los espacios aislados se inicializan de forma diferida en un nodo, la primera vez que una consulta requiere un espacio aislado para su ejecución.
    * Esto significa que la primera ejecución de un complemento que usa espacios aislados en un nodo incluirá un breve período de preparación.
  * Cuando se reinicia un nodo, por ejemplo, como parte de una actualización del servicio, se eliminan todos los espacios aislados que se están ejecutando en él.
* Cada nodo mantiene un número predefinido de espacios aislados que están listos para ejecutar solicitudes entrantes.
  * Una vez que se usa un espacio aislado, uno nuevo está disponible automáticamente para reemplazarlo.
* Si no hay ningún espacio aislado previamente asignado para atender a un operador de consulta, se limitará hasta que estén disponibles los nuevos espacios aislados. Para obtener más información, vea [errores](#errors). La nueva asignación de espacio aislado podría tardar hasta 10-15 segundos por espacio aislado, dependiendo de la SKU y los recursos disponibles en el nodo de datos.

## <a name="limitations"></a>Limitaciones

Algunas de las limitaciones se pueden controlar mediante una [Directiva de espacio aislado](../management/sandboxpolicy.md)en el nivel de clúster, para cada tipo de espacio aislado.

* **Número de espacios aislados por nodo:** El número de espacios aislados por nodo es limitado.
  * Las solicitudes que se realicen cuando no haya ningún espacio aislado disponible se limitarán.
* **Red:** Un espacio aislado no puede interactuar con ningún recurso de la máquina virtual (VM) o fuera de él.
  * Un espacio aislado no puede interactuar con otro espacio aislado.
* **CPU:** La velocidad máxima de CPU que puede consumir un espacio aislado de los procesadores de su host es limitada (el valor predeterminado es `50%` ).
  * Cuando se alcanza el límite, se limita el uso de CPU del espacio aislado, pero continúa la ejecución.
* **Memoria:** La cantidad máxima de memoria RAM que puede consumir un espacio aislado de la RAM del host es limitada (el valor predeterminado es `20GB` ).
  * Alcanzar el límite da como resultado la terminación del espacio aislado y un error de ejecución de la consulta.
* **Disco:** Un espacio aislado tiene un directorio único e independiente conectado. No puede tener acceso al sistema de archivos del host.
  * La carpeta única proporciona acceso a la imagen o paquete que coincide con el tipo de espacio aislado. Por ejemplo, el paquete de Python o R no personalizable.
* **Procesos secundarios:** El espacio aislado está bloqueado para generar procesos secundarios.

> [!NOTE]
> Los recursos usados con espacio aislado no dependen solo del tamaño de los datos que se procesan como parte de la solicitud, sino también de la lógica que se ejecuta en el espacio aislado y de la implementación de las bibliotecas que usa.
> Por ejemplo, para los `python` `r` Complementos y, el último significa el script proporcionado por el usuario y las bibliotecas de Python o R que utiliza en tiempo de ejecución.

## <a name="errors"></a>Errors

|Código                      |Message                                                                                        |Posible motivo                                                                                                    |
|--------------------------|-----------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|
|E_SB_QUERY_THROTTLED_ERROR|La consulta en espacio aislado se anuló debido a la limitación. Volver a intentar después de algún retroceso puede realizarse correctamente   |No hay ningún espacio aislado disponible en el nodo de destino. Los nuevos espacios aislados deben estar disponibles en unos segundos.     |
|E_SB_QUERY_THROTTLED_ERROR|Los espacios aislados del tipo ' {Kind} ' todavía no se han inicializado                       |La Directiva de espacio aislado ha cambiado recientemente. Los nuevos espacios aislados que obedecen a la nueva Directiva estarán disponibles en unos segundos.           |
