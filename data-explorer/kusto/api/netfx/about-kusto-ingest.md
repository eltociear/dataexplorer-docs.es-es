---
title: 'Biblioteca de cliente de introducción de Kusto: Azure Explorador de datos | Microsoft Docs'
description: En este artículo se describe la biblioteca de cliente de Kusto ingesta en Azure Explorador de datos.
services: data-explorer
author: orspod
ms.author: orspodek
ms.reviewer: rkarlin
ms.service: data-explorer
ms.topic: reference
ms.custom: has-adal-ref
ms.date: 03/24/2020
ms.openlocfilehash: 5770c59ff7298567cad01bb3ed4cc6a684b2378a
ms.sourcegitcommit: bb8c61dea193fbbf9ffe37dd200fa36e428aff8c
ms.translationtype: MT
ms.contentlocale: es-ES
ms.lasthandoff: 05/13/2020
ms.locfileid: "83373690"
---
# <a name="kusto-ingest-client-library"></a>Biblioteca de cliente de introducción de Kusto

## <a name="overview"></a>Información general
La biblioteca Kusto. ingesta es una biblioteca .NET 4.6.2 que permite enviar datos al servicio Kusto.
Kusto. ingesta tiene dependencias en las bibliotecas y los SDK siguientes:

* ADAL para la autenticación de AAD
* Cliente de Azure Storage

Los métodos de ingesta Kusto se definen mediante la interfaz [IKustoIngestClient](kusto-ingest-client-reference.md#interface-ikustoingestclient) y permiten la ingesta de datos de Stream, IDataReader, archivos locales y blobs de Azure en los modos sincrónico y asincrónico.

## <a name="ingest-client-flavors"></a>Introducción de los tipos de cliente
Conceptualmente, hay dos tipos básicos del cliente de introducción: en cola y directo.

### <a name="queued-ingestion"></a>Ingesta en cola
Definido por [IKustoQueuedIngestClient](kusto-ingest-client-reference.md#interface-ikustoqueuedingestclient), este modo limita la dependencia del código de cliente en el servicio Kusto. La ingesta se realiza mediante la publicación de un mensaje de ingesta de Kusto en una cola de Azure, que se adquiere a continuación desde el servicio de ingesta de Kusto Administración de datos. El cliente de introducción creará los artefactos de almacenamiento intermedio usando los recursos asignados por el servicio Administración de datos de Kusto.

Entre **las ventajas del modo en cola** se incluyen (pero no se limitan a):

* Desacoplamiento del proceso de ingesta de datos desde el servicio de motor de Kusto
* Permite que las solicitudes de ingesta se conserven cuando el servicio Kusto Engine (o ingesta) no esté disponible.
* Permite la agregación eficaz y controlable de datos de entrada por parte del servicio de ingesta, mejorando el rendimiento.
* Permite que el servicio de ingesta de Kusto administre la carga de ingesta en el servicio de motor de Kusto
* El servicio de ingesta de Kusto volverá a intentarlo según sea necesario en los errores de ingesta transitorios (por ejemplo, la limitación de XStore)
* Proporciona un mecanismo cómodo para realizar el seguimiento del progreso y el resultado de cada solicitud de ingesta.

En el diagrama siguiente se describe la interacción del cliente de ingesta en cola con Kusto:

![texto alternativo](../images/queued-ingest.jpg "en cola: ingesta")

### <a name="direct-ingestion"></a>Ingesta directa
Definido por IKustoDirectIngestClient, este modo fuerza la interacción directa con el servicio de motor de Kusto. En este modo, el servicio de ingesta de Kusto no modera ni administra los datos. Cada solicitud de ingesta en modo directo se traduce finalmente en el comando que se `.ingest` ejecuta directamente en el servicio de motor Kusto.
En el diagrama siguiente se describe la interacción directa del cliente de ingesta con Kusto:

![texto alternativo](../images/direct-ingest.jpg "ingesta directa")

> [!NOTE]
> El modo directo no se recomienda para las soluciones de ingesta de calidad de producción.

**Las ventajas del modo directo** incluyen:

* Latencia baja (no hay ninguna agregación). Sin embargo, la latencia baja también puede lograrse con la ingesta en cola.
* Cuando se usan métodos sincrónicos, la finalización de métodos indica el final de la operación de ingesta.

Entre **las desventajas del modo directo** se incluyen:

* El código de cliente debe implementar cualquier lógica de control de errores o de reintento
* No es posible la ingesta cuando el servicio de motor de Kusto no está disponible
* El código de cliente podría sobrecargar el servicio de motor Kusto con solicitudes de ingesta, ya que no es consciente de la capacidad del servicio de motor.

## <a name="ingestion-best-practices"></a>Prácticas recomendadas de ingesta

### <a name="general"></a>General
Los [procedimientos recomendados de ingesta](kusto-ingest-best-practices.md) proporcionan el POV y el rendimiento de la ingesta.

### <a name="thread-safety"></a>Seguridad para subprocesos
Las implementaciones de cliente de Kusto ingesta son seguras para subprocesos y están pensadas para reutilizarlas. No es necesario crear una instancia de la `KustoQueuedIngestClient` clase para cada una de las operaciones de introducción o incluso varias. Se requiere una instancia única de `KustoQueuedIngestClient` por cada proceso de clúster de Kusto de destino. Ejecutar varias instancias es un contador productivo y puede producir DoS Administración de datos clúster.

### <a name="supported-data-formats"></a>Formatos de datos compatibles
Al usar la ingesta nativa, si aún no lo está, cargue los datos en uno o varios Azure Storage BLOBs. Los formatos BLOB actualmente admitidos se documentan en el tema [formatos de datos admitidos](../../../ingestion-supported-formats.md) .

### <a name="schema-mapping"></a>Asignación de esquemas
Las [asignaciones de esquema](../../management/mappings.md) ayudan a enlazar los campos de datos de origen de forma determinista con las columnas de la tabla de destino.

## <a name="usage-and-further-reading"></a>Uso y lecturas adicionales

* Como se ha descrito anteriormente, la base recomendada para las soluciones de ingesta sostenible y a gran escala para Kusto debe ser **KustoQueuedIngestClient**.
* Para minimizar la carga innecesaria en el servicio Kusto, se recomienda que se use una única instancia del cliente de ingesta de Kusto (en cola o directo) por proceso por clúster de Kusto. La implementación del cliente de Kusto ingesta es segura para subprocesos y es totalmente reentrante.

### <a name="ingestion-permissions"></a>Permisos de ingesta
* [Permisos de ingesta de Kusto](kusto-ingest-client-permissions.md) explica la configuración de los permisos necesarios para una ingesta correcta mediante el paquete Kusto. ingesta

### <a name="kustoingest-library-reference"></a>Referencia de la biblioteca Kusto. ingesta
* La [referencia de cliente de Kusto. ingesta](kusto-ingest-client-reference.md) contiene una referencia completa de las interfaces e implementaciones de cliente de Kusto ingesta.<BR>Allí encontrará la información sobre cómo crear clientes de introducción, aumentar las solicitudes de ingesta, administrar el progreso de la ingesta y mucho más.
* [Estado de la operación Kusto. ingesta](kusto-ingest-client-status.md) explica los recursos de **KustoQueuedIngestClient** para el seguimiento del estado de ingesta
* [Kusto. ingesta de errores](kusto-ingest-client-errors.md) documentos Kusto ingesta de errores y excepciones de cliente
* Los [ejemplos de Kusto. ingesta](kusto-ingest-client-examples.md) presentan fragmentos de código que muestran diversas técnicas de introducción de datos en Kusto

### <a name="data-ingestion-rest-apis"></a>API de REST de ingesta de datos
La [ingesta de datos sin Kusto. la biblioteca de introducción](kusto-ingest-client-rest.md) explica cómo implementar la ingesta en cola Kusto mediante las API de REST de Kusto y sin tener que depender de la biblioteca Kusto. ingesta.
