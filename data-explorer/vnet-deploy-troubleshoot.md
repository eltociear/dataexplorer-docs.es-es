---
title: Solución de problemas de acceso, ingesta y funcionamiento del clúster de Azure Data Explorer en la red virtual
description: Solución de problemas de conectividad, ingesta, creación y operación del clúster de Azure Data Explorer en la red virtual
author: basaba
ms.author: basaba
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 03/24/2020
ms.openlocfilehash: b50b971a3b1980ad35a1a939bdf25f1c9e6ac7ba
ms.sourcegitcommit: 47a002b7032a05ef67c4e5e12de7720062645e9e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/15/2020
ms.locfileid: "81494004"
---
# <a name="troubleshoot-access-ingestion-and-operation-of-your-azure-data-explorer-cluster-in-your-virtual-network"></a>Solución de problemas de acceso, ingesta y funcionamiento del clúster de Azure Data Explorer en la red virtual

En esta sección aprenderá a solucionar problemas de conectividad, de operación y de creación de clústeres para un clúster que se implementa en la [red virtual](/azure/virtual-network/virtual-networks-overview).

## <a name="access-issues"></a>Problemas de acceso

Si tiene un problema al acceder al clúster mediante el punto de conexión público (cluster.region.kusto.windows.net) o privado (private-cluster.region.kusto.windows.net) y sospecha que está relacionado con la configuración de la red virtual, lleve a cabo los pasos siguientes para solucionar el problema.

### <a name="check-tcp-connectivity"></a>Comprobación de la conectividad TCP

El primer paso incluye la comprobación de la conectividad TCP mediante el sistema operativo Windows o Linux.

# <a name="windows"></a>[Windows](#tab/windows)

   1. Descargue [TCping](https://www.elifulkerson.com/projects/tcping.php) en la máquina que se conecta al clúster.
   2. Haga ping al destino desde la máquina de origen con el siguiente comando:

    ```cmd
     C:\> tcping -t yourcluster.kusto.windows.net 443 
    
     ** Pinging continuously.  Press control-c to stop **
    
     Probing 1.2.3.4:443/tcp - Port is open - time=100.00ms
     ```

# <a name="linux"></a>[Linux](#tab/linux)

   1. Instale *netcat* en la máquina que se conecta al clúster.

    ```bash
    $ apt-get install netcat
     ```

   2. Haga ping al destino desde la máquina de origen con el siguiente comando:

     ```bash
     $ netcat -z -v yourcluster.kusto.windows.net 443
    
     Connection to yourcluster.kusto.windows.net 443 port [tcp/https] succeeded!
     ```
---

Si la prueba no finaliza correctamente, continúe con los pasos siguientes. Si la prueba es correcta, el problema no se debe a la conectividad de TCP. Vaya a [Problemas operativos](#cluster-creation-and-operations-issues) para continuar con la solución del problema.

### <a name="check-the-network-security-group-nsg"></a>Comprobación del grupo de seguridad de red (NSG)

   Compruebe que el [grupo de seguridad de red](/azure/virtual-network/security-overview) (NSG) conectado a la subred del clúster tiene una regla de entrada que permite el acceso desde la dirección IP del equipo cliente al puerto 443.

### <a name="check-route-table"></a>Comprobación de la tabla de rutas

   Si la subred del clúster tiene la configuración de tunelización forzada en el firewall (subred con una [tabla de rutas](/azure/virtual-network/virtual-networks-udr-overview) que contiene la ruta predeterminada "0.0.0.0/0"), asegúrese de que la dirección IP de la máquina tenga una ruta con el [tipo de próximo salto](/azure/virtual-network/virtual-networks-udr-overview) establecido en VirtualNetwork/Internet. Esta ruta se necesita para evitar problemas de rutas asimétricas.

## <a name="ingestion-issues"></a>Problemas de ingesta

Si está experimentando problemas de ingesta y sospecha que están relacionados con la configuración de la red virtual, lleve a cabo los pasos siguientes.

### <a name="check-ingestion-health"></a>Comprobación del estado de la ingesta

Comprueba que las [métricas de ingesta de clúster](/azure/data-explorer/using-metrics#ingestion-health-and-performance-metrics) indiquen un estado correcto.

### <a name="check-security-rules-on-data-source-resources"></a>Comprobación de las reglas de seguridad en los recursos de origen de datos

Si las métricas indican que no se procesó ningún evento desde el origen de datos (*Eventos procesados* métrica para instancias de Event Hub o IoT Hub), asegúrese de que los recursos del origen de datos (Event Hub o Storage) permiten el acceso desde la subred del clúster en las reglas de firewall o los puntos de conexión de servicio.

### <a name="check-security-rules-configured-on-clusters-subnet"></a>Comprobación de las reglas de seguridad configuradas en la subred del clúster

Asegúrese de que la subred del clúster tiene las reglas de NSG, UDR y firewall configuradas correctamente. Además, pruebe la conectividad de red para todos los puntos de conexión dependientes. 

## <a name="cluster-creation-and-operations-issues"></a>Problemas de creación y operación de clústeres

Si experimenta problemas con la creación o la operación de un clúster y sospecha que está relacionado con la configuración de la red virtual, siga estos pasos para solucionar el problema.

### <a name="diagnose-the-virtual-network-with-the-rest-api"></a>Diagnóstico de la red virtual con la API REST

Se usa [ARMClient](https://chocolatey.org/packages/ARMClient) para llamar a la API REST con PowerShell. 

1. Inicio de sesión con ARMClient

   ```powerShell
   armclient login
   ```

1. Invoque la operación de diagnóstico.

    ```powershell
    $subscriptionId = '<subscription id>'
    $clusterName = '<name of cluster>'
    $resourceGroupName = '<resource group name>'
    $apiversion = '2019-11-09'
    
    armclient post "https://management.azure.com/subscriptions/$subscriptionId/resourceGroups/$resourceGroupName/providers/Microsoft.Kusto/clusters/$clusterName/diagnoseVirtualNetwork?api-version=$apiversion" -verbose
    ```

1. Compruebe la respuesta.

    ```powershell
    HTTP/1.1 202 Accepted
    ...
    Azure-AsyncOperation: https://management.azure.com/subscriptions/{subscription-id}/providers/Microsoft.Kusto/locations/{location}/operationResults/{operation-id}?api-version=2019-11-09
    ...
    ```

1. Espere a que finalice la operación.

    ```powershell
    armclient get https://management.azure.com/subscriptions/$subscriptionId/providers/Microsoft.Kusto/locations/{location}/operationResults/{operation-id}?api-version=2019-11-09
    
    {
      "id": "/subscriptions/{subscription-id}/providers/Microsoft.Kusto/locations/{location}/operationresults/{operation-id}",
      "name": "{operation-name}",
      "status": "[Running/Failed/Completed]",
      "startTime": "{start-time}",
      "endTime": "{end-time}",
      "properties": {...}
    }
    ```
    
   Espere hasta que la propiedad *status* muestre *Completed*; después, el campo *properties* debe mostrar:

    ```powershell
    {
      "id": "/subscriptions/{subscription-id}/providers/Microsoft.Kusto/locations/{location}/operationresults/{operation-id}",
      "name": "{operation-name}",
      "status": "Completed",
      "startTime": "{start-time}",
      "endTime": "{end-time}",
      "properties": {
        "Findings": [...]
      }
    }
    ```

Si la propiedad *Findings* muestra un resultado vacío, significa que se han superado todas las pruebas de red y no se ha interrumpido la conexión. Si se muestra el siguiente error, *Outbound dependency '{dependencyName}:{port}' might be not satisfied (Outbound)* (Puede que no se haya satisfecho la dependencia de salida '{nombreDeDependencia}:{puerto}' [salida]), el clúster no puede acceder a los puntos de conexión de servicio dependientes. Siga los pasos que se indican a continuación.

### <a name="check-network-security-group-nsg"></a>Comprobación del grupo de seguridad de red (NSG)

Asegúrese de que el [grupo de seguridad de red](/azure/virtual-network/security-overview) esté configurado correctamente según las instrucciones de [Dependencias para la implementación de redes virtuales](/azure/data-explorer/vnet-deployment#dependencies-for-vnet-deployment).

### <a name="check-route-table"></a>Comprobación de la tabla de rutas

Si la subred del clúster tiene configurada la tunelización forzada en el firewall (subred con una [tabla de rutas](/azure/virtual-network/virtual-networks-udr-overview) que contiene la ruta predeterminada "0.0.0.0/0"), asegúrese de que las [direcciones IP de administración](vnet-deployment.md#azure-data-explorer-management-ip-addresses)) y las [direcciones IP de supervisión de estado](vnet-deployment.md#health-monitoring-addresses) tienen una ruta con el [tipo de próximo salto](/azure/virtual-network/virtual-networks-udr-overview##next-hop-types-across-azure-tools) establecido en *Internet* y un [prefijo de dirección de origen](/azure/virtual-network/virtual-networks-udr-overview#how-azure-selects-a-route) en *"management-ip/32"* y *"health-monitoring-ip/32"* . Esta ruta es necesaria para evitar problemas de rutas asimétricas.

### <a name="check-firewall-rules"></a>Comprobación de las reglas de firewall

Si fuerza la tunelización del tráfico de salida de la subred a un firewall, asegúrese de que en la configuración del firewall se permiten todas las dependencias FQDN (por ejemplo, *.blob.core.windows.net*), tal como se describe en [Protección del tráfico de salida con el firewall](/azure/data-explorer/vnet-deployment#securing-outbound-traffic-with-firewall).
