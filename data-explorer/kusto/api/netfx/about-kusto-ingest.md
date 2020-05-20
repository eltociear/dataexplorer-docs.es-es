---
title: 'Biblioteca de cliente de introducción de Kusto: Azure Explorador de datos'
description: En este artículo se describe la biblioteca de cliente de Kusto ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
ms.date: 03/18/2020
ms.openlocfilehash: a5655ac982ab65d06cdee7c93a166dce51377eed
ms.sourcegitcommit: e66c5f4b833b4f6269bb7bfa5695519fcb11d9fa
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/19/2020
ms.locfileid: "83630039"
---
# <a name="kusto-ingest-client-library"></a>Biblioteca de cliente de introducción de Kusto 

`Kusto.Ingest`Biblioteca es una biblioteca .NET 4.6.2 para enviar datos al servicio Kusto.
Toma dependencias en las bibliotecas y los SDK siguientes:

* ADAL para la autenticación de Azure AD
* Cliente de Azure Storage

La interfaz [IKustoIngestClient](kusto-ingest-client-reference.md#interface-ikustoingestclient) define los métodos de ingesta.  Los métodos controlan la ingesta de datos de Stream, IDataReader, archivos locales y blobs de Azure en modos sincrónicos y asincrónicos.

## <a name="ingest-client-flavors"></a>Introducción de los tipos de cliente

Hay dos tipos básicos del cliente de introducción: en cola y directo.

### <a name="queued-ingestion"></a>Ingesta en cola

El modo de ingesta en cola, definido por [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient), limita la dependencia del código de cliente en el servicio Kusto. La ingesta se realiza mediante la publicación de un mensaje de ingesta de Kusto en una cola de Azure, que se adquiere después del servicio Kusto Administración de datos (ingesta). El cliente de introducción creará cualquier elemento de almacenamiento intermedio usando los recursos del servicio Administración de datos de Kusto.

**Entre las ventajas del modo en cola se incluyen las siguientes:**

* Desacopla el proceso de ingesta de datos del servicio de motor de Kusto
* Permite que las solicitudes de ingesta se conserven cuando el servicio del motor de Kusto (o la ingesta) no esté disponible.
* Mejora el rendimiento mediante la agregación eficaz y controlable de datos de entrada por parte del servicio de ingesta. 
* Permite que el servicio de ingesta de Kusto administre la carga de ingesta en el servicio de motor de Kusto
* Reintenta el servicio de ingesta de Kusto, según sea necesario, en errores de ingesta transitorios, como para la limitación de XStore
* Proporciona un mecanismo cómodo para realizar el seguimiento del progreso y el resultado de cada solicitud de ingesta.

En el diagrama siguiente se describe la interacción del cliente de ingesta en cola con Kusto:

:::image type="content" source="../images/about-kusto-ingest/queued-ingest.png" alt-text="en cola: ingesta":::
 
### <a name="direct-ingestion"></a>Ingesta directa

El modo de ingesta directo, definido por IKustoDirectIngestClient, fuerza la interacción directa con el servicio de motor de Kusto. En este modo, el servicio de ingesta de Kusto no modera ni administra los datos. Cada solicitud de ingesta se traduce finalmente en el `.ingest` comando que se ejecuta directamente en el servicio de motor Kusto.

En el diagrama siguiente se describe la interacción directa del cliente de ingesta con Kusto:

:::image type="content" source="../images/about-kusto-ingest/direct-ingest.png" alt-text="ingesta directa":::

> [!NOTE]
> No se recomienda el modo directo para las soluciones de ingesta de calidad de producción.

**Las ventajas del modo directo incluyen:**

* Baja latencia y sin agregación. Sin embargo, la latencia baja también puede lograrse con la ingesta en cola.
* Cuando se usan métodos sincrónicos, la finalización de métodos indica el final de la operación de ingesta.

**Entre las desventajas del modo directo se incluyen:**

* El código de cliente debe implementar cualquier lógica de control de errores o de reintento
* No es posible la ingesta cuando el servicio de motor de Kusto no está disponible
* El código de cliente podría sobrecargar el servicio de motor Kusto con solicitudes de ingesta, ya que no es consciente de la capacidad del servicio de motor

## <a name="ingestion-best-practices"></a>Prácticas recomendadas de ingesta

Los [procedimientos recomendados de ingesta](kusto-ingest-best-practices.md) proporcionan el POV y el rendimiento de la ingesta.

* **Seguridad para subprocesos:** Las implementaciones de cliente de Kusto ingesta son seguras para subprocesos y están pensadas para reutilizarlas. No es necesario crear una instancia de la `KustoQueuedIngestClient` clase para cada una de las operaciones de introducción o varias. Se requiere una instancia única de `KustoQueuedIngestClient` por cada proceso de clúster de Kusto de destino. Ejecutar varias instancias es un contador productivo y puede provocar DoS en el clúster de Administración de datos.

* **Formatos de datos admitidos:** Al usar la ingesta nativa, si aún no lo está, cargue los datos en uno o varios blobs de Azure Storage. Los formatos BLOB actualmente admitidos están documentados en [formatos de datos admitidos](../../../ingestion-supported-formats.md).

* **Asignación de esquemas:** 
 Las [asignaciones de esquema](../../management/mappings.md) ayudan a enlazar los campos de datos de origen de forma determinista con las columnas de la tabla de destino.

* **Permisos de ingesta:** 
 [Los permisos de ingesta de Kusto](kusto-ingest-client-permissions.md) explican la configuración de permisos necesaria para una ingesta correcta mediante el `Kusto.Ingest` paquete.

* **Uso:** Tal y como se ha descrito anteriormente, la base recomendada para las soluciones de ingesta sostenibles y a gran escala para Kusto debe ser **KustoQueuedIngestClient**.
Para minimizar la carga innecesaria en el servicio Kusto, se recomienda usar una única instancia del cliente de ingesta de Kusto (en cola o directo) por proceso, por clúster de Kusto. La implementación del cliente de Kusto ingesta es segura para subprocesos y es totalmente reentrante.

## <a name="next-steps"></a>Pasos siguientes

* La [referencia de cliente de Kusto. ingesta](kusto-ingest-client-reference.md) contiene una referencia completa de las interfaces e implementaciones de cliente de Kusto ingesta. Encontrará información sobre cómo crear clientes de ingesta, aumentar las solicitudes de ingesta, administrar el progreso de la ingesta, etc.

* [Estado de la operación Kusto. ingesta](kusto-ingest-client-status.md) explica las características de KustoQueuedIngestClient para el seguimiento del estado de ingesta

* Los [errores de Kusto. ingesta](kusto-ingest-client-errors.md) describen errores y excepciones del cliente de ingesta de Kusto

* Los [ejemplos de Kusto. ingesta](kusto-ingest-client-examples.md) muestran fragmentos de código que muestran diversas técnicas de introducción de datos en Kusto

* [Ingesta de datos sin Kusto. la biblioteca de introducción](kusto-ingest-client-rest.md) explica cómo implementar la ingesta en cola de Kusto, mediante el uso de las API de REST de Kusto y sin depender de la `Kusto.Ingest` biblioteca.
