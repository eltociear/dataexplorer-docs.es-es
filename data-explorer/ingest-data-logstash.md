---
title: Ingesta de datos de Logstash en Azure Data Explorer
description: En este artículo obtendrá información sobre cómo ingerir (cargar) datos en Azure Data Explorer desde Logstash
author: tamirkamara
ms.author: takamara
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 06/03/2019
ms.openlocfilehash: e7b4a8cf3dcd109de59ab12717f8fbc17b600ec9
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81493140"
---
# <a name="ingest-data-from-logstash-to-azure-data-explorer"></a>Ingesta de datos de Logstash en Azure Data Explorer

[Logstash](https://www.elastic.co/products/logstash) es una canalización de procesamiento de datos del servidor de código abierto que ingiere datos de varios orígenes al mismo tiempo, los transforma y, después, los envía a su "lugar de almacenamiento provisional" preferido. En este artículo, enviará dichos datos a Azure Data Explorer, que es un servicio de exploración de datos rápido y muy escalable para datos de telemetría y registro. Inicialmente creará una tabla y una asignación de datos en un clúster de prueba y, después, hará que Logstash envíe los datos a la tabla y valide los resultados.

## <a name="prerequisites"></a>Prerequisites

* Suscripción a Azure. Si no la tiene, [cree una cuenta gratuita de Azure](https://azure.microsoft.com/free/) antes de empezar.
* Un clúster y una base de datos de prueba de [Azure Data Explorer](create-cluster-database-portal.md).
* [Instrucciones de instalación](https://www.elastic.co/guide/en/logstash/current/installing-logstash.html) de la versión 6, o posteriores, de Logstash

## <a name="create-a-table"></a>Creación de una tabla

Una vez que tenga un clúster y una base de datos, es el momento de crear una tabla.

1. Ejecute el siguiente comando en la ventana de consulta de la base de datos para crear una tabla:

    ```Kusto
    .create table logs (timestamp: datetime, message: string)
    ```

1. Ejecute el siguiente comando para confirmar que la nueva tabla `logs` se ha creado y que está vacía:
    ```Kusto
    logs
    | count
    ```

## <a name="create-a-mapping"></a>Creación de una asignación

Azure Data Explorer utiliza la asignación para transformar los datos entrantes en el esquema de tabla de destino. El siguiente comando crea una nueva asignación denominada `basicmsg` que extrae las propiedades del archivo JSON entrante que se indican mediante el `path` y las envía a `column`.

Ejecute el siguiente comando en la ventana de consulta:

```Kusto
.create table logs ingestion json mapping 'basicmsg' '[{"column":"timestamp","path":"$.@timestamp"},{"column":"message","path":"$.message"}]'
```

## <a name="install-the-logstash-output-plugin"></a>Instalación del complemento de salida de Logstash

El complemento de salida de Logstash se comunica con Azure Data Explorer y envía los datos al servicio.
Para instalarlo, ejecute el siguiente comando en el directorio raíz de Logstash:

```sh
bin/logstash-plugin install logstash-output-kusto
```

## <a name="configure-logstash-to-generate-a-sample-dataset"></a>Configuración de Logstash para generar un conjunto de datos de ejemplo

Logstash puede generar eventos de ejemplo que pueden utilizarse para probar una canalización de un extremo a otro.
Si ya utiliza Logstash y tiene acceso a su propio flujo de eventos, vaya a la sección siguiente. 

> [!NOTE]
> Si usa sus propios datos, cambie la tabla y los objetos de asignación que se han definido en los pasos anteriores.

1. Edite un nuevo archivo de texto que contendrá la configuración necesaria de la canalización (mediante vi):

    ```sh
    vi test.conf
    ```

1. Pegue la siguiente configuración, que indicará a Logstash que genere 1000 eventos de prueba:

    ```ruby
    input {
        stdin { }
        generator {
            message => "Test Message 123"
            count => 1000
        }
    }
    ```

Esta configuración también incluye el complemento de entrada `stdin`, que le permitirá escribir más mensajes (no olvide usar *Entrar* para enviarlos a la canalización).

## <a name="configure-logstash-to-send-data-to-azure-data-explorer"></a>Configuración de Logstash para enviar datos a Azure Data Explorer

Pegue la siguiente configuración en el archivo de configuración que usó en el paso anterior. Reemplace todos los marcadores de posición por los valores pertinente para la configuración. Para más información, consulte [Creating an AAD Application](kusto/management/access-control/how-to-provision-aad-app.md) (Creación de una aplicación AAD). 

```ruby
output {
    kusto {
            path => "/tmp/kusto/%{+YYYY-MM-dd-HH-mm-ss}.txt"
            ingest_url => "https://ingest-<cluster name>.kusto.windows.net/"
            app_id => "<application id>"
            app_key => "<application key/secret>"
            app_tenant => "<tenant id>"
            database => "<database name>"
            table => "<target table>" # logs as defined above
            mapping => "<mapping name>" # basicmsg as defined above
    }
}
```

| Nombre de parámetro | Descripción |
| --- | --- |
| **path** | El complemento de Logstash escribe eventos en archivos temporales antes de enviarlos a Azure Data Explorer. Este parámetro incluye una ruta de acceso en la que se deben escribir los archivos y una expresión de tiempo para la rotación de archivos desencadenará una carga en el servicio Azure Data Explorer.|
| **ingest_url** | El punto de conexión de Kusto para las comunicaciones relacionadas con la ingesta.|
| **app_id**,  **app_key** y **app_tenant**| Credenciales necesarias para conectarse a Azure Data Explorer. Asegúrese de usar una aplicación con privilegios de ingesta. |
| **database**| Nombre de la base de datos en la que se colocan los eventos. |
| **table** | Nombre de la tabla de destino en la que se colocan los eventos. |
| **mapping** | La asignación se usa para asignar a una cadena json de eventos entrantes el formato de fila correcto (define la propiedad que va en cada columna). |

## <a name="run-logstash"></a>Ejecución de Logstash

Ya estamos preparados ejecutar Logstash y probar nuestra configuración.

1. En el directorio raíz de Logstash, ejecute el siguiente comando:

    ```sh
    bin/logstash -f test.conf
    ```

    Debería ver la información impresa en la pantalla y, después, los 1000 mensajes que ha generados nuestra configuración de ejemplo. En este momento, también puede escribir más mensajes manualmente.

1. Después de unos minutos, ejecute la siguiente consulta de Data Explorer para ver los mensajes en la tabla que ha definido:

    ```Kusto
    logs
    | order by timestamp desc
    ```

1. Seleccione Ctrl + C para salir de Logstash

## <a name="clean-up-resources"></a>Limpieza de recursos

Ejecute el siguiente comando en la base de datos para limpiar la tabla `logs`:

```Kusto
.drop table logs
```

## <a name="next-steps"></a>Pasos siguientes

* [Escritura de consultas](write-queries.md)