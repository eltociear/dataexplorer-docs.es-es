---
title: Implementación de Azure Data Explorer en la red virtual
description: Obtenga información sobre cómo implementar Azure Data Explorer en la red virtual
author: basaba
ms.author: basaba
ms.reviewer: orspodek
ms.service: data-explorer
ms.topic: conceptual
ms.date: 10/31/2019
ms.openlocfilehash: 170db0d00f42209a2fd55c4009ab2f11eff55641
ms.sourcegitcommit: c4aea69fafa9d9fbb814764eebbb0ae93fa87897
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/17/2020
ms.locfileid: "81610207"
---
# <a name="deploy-azure-data-explorer-cluster-into-your-virtual-network"></a>Implementación del clúster Azure Data Explorer en la red virtual

En este artículo se explican los recursos que están presentes cuando se implementa un clúster de Azure Data Explorer en una red virtual de Azure personalizada. Esta información le ayudará a implementar un clúster en una subred de la red virtual (VNet). Para más información sobre las redes virtuales de Azure, vea [¿Qué es Azure Virtual Network?](/azure/virtual-network/virtual-networks-overview)

   ![Diagrama de red virtual](media/vnet-deployment/vnet-diagram.png)

Azure Data Explorer admite la implementación de un clúster en una subred de la red virtual (VNet). Esta funcionalidad le permite:

* Aplicar reglas de [grupo de seguridad de red](/azure/virtual-network/security-overview) (NSG) al tráfico del clúster de Azure Data Explorer.
* Conectar la red local a la subred del clúster de Azure Data Explorer.
* Proteger los orígenes de conexión de datos ([Event Hub](/azure/event-hubs/event-hubs-about) y [Event Grid](/azure/event-grid/overview)) con [puntos de conexión de servicio](/azure/virtual-network/virtual-network-service-endpoints-overview).

## <a name="access-your-azure-data-explorer-cluster-in-your-vnet"></a>Acceso al clúster de Azure Data Explorer en la red virtual

Puede acceder al clúster de Azure Data Explorer mediante las siguientes direcciones IP para cada servicio (servicios de administración de datos y motor):

* **Dirección IP privada**: se usa para acceder al clúster dentro de la red virtual.
* **Dirección IP pública**: se usa para acceder al clúster desde fuera de la red virtual para la administración y supervisión, y como una dirección de origen para las conexiones salientes iniciadas desde el clúster.

Se crean los siguientes registros DNS para acceder al servicio: 

* `[clustername].[geo-region].kusto.windows.net` (motor) `ingest-[clustername].[geo-region].kusto.windows.net` (administración de datos) se asignan a la dirección IP pública de cada servicio. 

* `private-[clustername].[geo-region].kusto.windows.net` (motor) `private-ingest-[clustername].[geo-region].kusto.windows.net` (administración de datos) se asignan a la dirección IP privada de cada servicio.

## <a name="plan-subnet-size-in-your-vnet"></a>Planeamiento del tamaño de la subred en la red virtual

El tamaño de la subred que se usa para hospedar un clúster de Azure Data Explorer no se puede modificar una vez implementada la subred. En la red virtual, Azure Data Explorer usa una dirección IP privada para cada máquina virtual y dos direcciones IP privadas para los equilibradores de carga internos (motor y administración de datos). Las redes de Azure también usan cinco direcciones IP para cada subred. Azure Data Explorer aprovisiona dos máquinas virtuales para el servicio de administración de datos. Las máquinas virtuales del servicio de motor se aprovisionan por capacidad de escala de configuración de usuario.

El número total de direcciones IP:

| Uso | Número de direcciones |
| --- | --- |
| Servicio de motor | 1 por instancia |
| Servicio de administración de datos | 2 |
| Equilibradores de carga internos | 2 |
| Direcciones reservadas de Azure | 5 |
| **Total** | **#instancias_de_motor + 9** |

> [!IMPORTANT]
> El tamaño de la subred se debe planear con antelación, ya que no se puede cambiar una vez implementado Azure Data Explorer. Por tanto, reserve el tamaño de subred necesario en consecuencia.

## <a name="service-endpoints-for-connecting-to-azure-data-explorer"></a>Puntos de conexión de servicio para conectarse a Azure Data Explorer

[Los puntos de conexión de servicio de Azure](/azure/virtual-network/virtual-network-service-endpoints-overview) permiten proteger los recursos multiinquilino de Azure en la red virtual.
La implementación del clúster de Azure Data Explorer en la subred permite configurar conexiones de datos con [Event Hub](/azure/event-hubs/event-hubs-about) o [Event Grid](/azure/event-grid/overview) mientras se restringen los recursos subyacentes de la subred de Azure Data Explorer.

> [!NOTE]
> Al usar la configuración de EventGrid con [Storage](/azure/storage/common/storage-introduction) y Event Hub, la cuenta de almacenamiento que se usa en la suscripción puede bloquearse con puntos de conexión de servicio a la subred de Azure Data Explorer, a la vez que permite los servicios de la plataforma de Azure de confianza en la [configuración de firewall](/azure/storage/common/storage-network-security). Sin embargo, Event Hub no puede habilitar el punto de conexión de servicio porque no admite [servicios de plataforma de Azure](/azure/event-hubs/event-hubs-service-endpoints) de confianza.

## <a name="dependencies-for-vnet-deployment"></a>Dependencias para la implementación de redes virtuales

### <a name="network-security-groups-configuration"></a>Configuración de grupos de seguridad de red

Los [grupos de seguridad de red (NSG)](/azure/virtual-network/security-overview) proporcionan la capacidad de controlar el acceso a la red dentro de una red virtual. Se puede acceder a Azure Data Explorer mediante dos puntos de conexión: HTTPs (443) y TDS (1433). Se deben configurar las reglas de NSG siguientes para permitir el acceso a estos puntos de conexión para la administración, la supervisión y el funcionamiento correcto del clúster.

#### <a name="inbound-nsg-configuration"></a>Configuración de NSG de entrada

| **Uso**   | **From**   | **To**   | **Protocolo**   |
| --- | --- | --- | --- |
| Administración  |[Direcciones de administración de ADX](#azure-data-explorer-management-ip-addresses)/AzureDataExplorerManagement(ServiceTag) | Subred de ADX: 443  | TCP  |
| Supervisión del estado  | [Direcciones de supervisión de estado de ADX](#health-monitoring-addresses)  | Subred de ADX: 443  | TCP  |
| Comunicación interna ADX  | Subred de ADX: Todos los puertos  | Subred de ADX: todos los puertos  | All  |
| Permitir entrada de Azure Load Balancer (sondeo de estado)  | AzureLoadBalancer  | Subred de ADX: 80,443  | TCP  |

#### <a name="outbound-nsg-configuration"></a>Configuración de NSG de salida

| **Uso**   | **From**   | **To**   | **Protocolo**   |
| --- | --- | --- | --- |
| Dependencia de Azure Storage  | Subred de ADX  | Almacenamiento: 443  | TCP  |
| Dependencia de Azure Data Lake  | Subred de ADX  | AzureDataLake: 443  | TCP  |
| Ingesta y supervisión de servicios de EventHub  | Subred de ADX  | EventHub: 443,5671  | TCP  |
| Publicación de métricas  | Subred de ADX  | AzureMonitor: 443 | TCP  |
| Descarga de la configuración de Azure Monitor  | Subred de ADX  | [Direcciones de punto de conexión de configuración de Azure Monitor](#azure-monitor-configuration-endpoint-addresses): 443 | TCP  |
| Active Directory (si es aplicable) | Subred de ADX | AzureActiveDirectory: 443 | TCP |
| Entidad de certificación | Subred de ADX | Internet: 80 | TCP |
| Comunicación interna  | Subred de ADX  | Subred de ADX: todos los puertos  | All  |
| Puertos que se usan para los complementos `sql\_request` y `http\_request`  | Subred de ADX  | Internet: personalizado  | TCP  |

### <a name="relevant-ip-addresses"></a>Direcciones IP pertinentes

#### <a name="azure-data-explorer-management-ip-addresses"></a>Direcciones IP de administración de Azure Data Explorer

| Region | Direcciones |
| --- | --- |
| Centro de Australia | 20.37.26.134 |
| Centro de Australia 2 | 20.39.99.177 |
| Este de Australia | 40.82.217.84 |
| Sudeste de Australia | 20.40.161.39 |
| BrazilSouth | 191.233.25.183 |
| Centro de Canadá | 40.82.188.208 |
| Este de Canadá | 40.80.255.12 |
| Centro de la India | 40.81.249.251, 104.211.98.159 |
| Centro de EE. UU. | 40.67.188.68 |
| EUAP del centro de EE. UU. | 40.89.56.69 |
| Este de Asia | 20.189.74.103 |
| Este de EE. UU. | 52.224.146.56 |
| Este de EE. UU. 2 | 52.232.230.201 |
| EUAP de Este de EE. UU. 2 | 52.253.226.110 |
| Centro de Francia | 40.66.57.91 |
| Sur de Francia | 40.82.236.24 |
| Japón Oriental | 20.43.89.90 |
| Japón Occidental | 40.81.184.86 |
| Centro de Corea del Sur | 40.82.156.149 |
| Corea del Sur | 40.80.234.9 |
| Centro-Norte de EE. UU | 40.81.45.254 |
| Norte de Europa | 52.142.91.221 |
| Norte de Sudáfrica | 102.133.129.138 |
| Oeste de Sudáfrica | 102.133.0.97 |
| Centro-sur de EE. UU. | 20.45.3.60 |
| Sudeste de Asia | 40.119.203.252 |
| Sur de la India | 40.81.72.110, 104.211.224.189 |
| Sur de Reino Unido 2 | 40.81.154.254 |
| Oeste de Reino Unido | 40.81.122.39 |
| Centro-Oeste de EE. UU. | 52.159.55.120 |
| Oeste de Europa | 51.145.176.215 |
| Oeste de la India | 40.81.88.112, 104.211.160.120 |
| Oeste de EE. UU. | 13.64.38.225 |
| Oeste de EE. UU. 2 | 40.90.219.23 |

#### <a name="health-monitoring-addresses"></a>Direcciones de supervisión de estado

| Region | Direcciones |
| --- | --- |
| Centro de Australia | 191.239.64.128 |
| Centro de Australia 2 | 191.239.64.128 |
| Este de Australia | 191.239.64.128 |
| Sudeste de Australia | 191.239.160.47 |
| Sur de Brasil | 23.98.145.105 |
| Centro de Canadá | 168.61.212.201 |
| Este de Canadá | 168.61.212.201 |
| Centro de la India | 23.99.5.162 |
| Centro de EE. UU. | 168.61.212.201 |
| EUAP del centro de EE. UU. | 168.61.212.201 |
| Este de Asia | 168.63.212.33 |
| Este de EE. UU. | 137.116.81.189 |
| Este de EE. UU. 2 | 137.116.81.189 |
| EUAP de Este de EE. UU. 2 | 137.116.81.189 |
| Centro de Francia | 23.97.212.5 |
| Sur de Francia | 23.97.212.5 |
| Japón Oriental | 138.91.19.129 |
| Japón Occidental | 138.91.19.129 |
| Centro de Corea del Sur | 138.91.19.129 |
| Corea del Sur | 138.91.19.129 |
| Centro-Norte de EE. UU | 23.96.212.108 |
| Norte de Europa | 191.235.212.69 
| Norte de Sudáfrica | 104.211.224.189 |
| Oeste de Sudáfrica | 104.211.224.189 |
| Centro-sur de EE. UU. | 23.98.145.105 |
| Sur de la India | 23.99.5.162 |
| Sudeste de Asia | 168.63.173.234 |
| Sur de Reino Unido 2 | 23.97.212.5 |
| Oeste de Reino Unido | 23.97.212.5 |
| Centro-Oeste de EE. UU. | 168.61.212.201 |
| Oeste de Europa | 23.97.212.5 |
| Oeste de la India | 23.99.5.162 |
| Oeste de EE. UU. | 23.99.5.162 |
| Oeste de EE. UU. 2 | 23.99.5.162, 104.210.32.14 |    

#### <a name="azure-monitor-configuration-endpoint-addresses"></a>Direcciones de punto de conexión de configuración de Azure Monitor

| Region | Direcciones |
| --- | --- |
| Centro de Australia | 52.148.86.165 |
| Centro de Australia 2 | 52.148.86.165 |
| Este de Australia | 52.148.86.165 |
| Sudeste de Australia | 52.148.86.165 |
| Sur de Brasil | 13.68.89.19 |
| Centro de Canadá | 13.90.43.231 |
| Este de Canadá | 13.90.43.231 |
| India Central | 13.71.25.187 |
| Centro de EE. UU. | 52.173.95.68 |
| EUAP de centro de EE. UU. | 13.90.43.231 |
| Asia Oriental | 13.75.117.221 |
| Este de EE. UU. | 13.90.43.231 |
| Este de EE. UU. 2 | 13.68.89.19 |    
| EUAP de Este de EE. UU. 2 | 13.68.89.19 |
| Centro de Francia | 52.174.4.112 |
| Sur de Francia | 52.174.4.112 |
| Este de Japón | 13.75.117.221 |
| Oeste de Japón | 13.75.117.221 |
| Centro de Corea del Sur | 13.75.117.221 |
| Sur de Corea del Sur | 13.75.117.221 |
| Centro-norte de EE. UU. | 52.162.240.236 |
| Norte de Europa | 52.169.237.246 |
| Norte de Sudáfrica | 13.71.25.187 |
| Oeste de Sudáfrica | 13.71.25.187 |
| Centro-sur de EE. UU. | 13.84.173.99 |
| India del Sur | 13.71.25.187 |
| Sudeste Asiático | 52.148.86.165 |
| Sur de Reino Unido | 52.174.4.112 |
| Oeste de Reino Unido | 52.169.237.246 |
| Centro-Oeste de EE. UU. | 52.161.31.69 |
| Oeste de Europa | 52.174.4.112 |
| Oeste de la India | 13.71.25.187 |
| Oeste de EE. UU. | 40.78.70.148 |
| Oeste de EE. UU. 2 | 52.151.20.103 |

## <a name="expressroute-setup"></a>Configuración de ExpressRoute

Use ExpressRoute para conectar la red local a Azure Virtual Network. Una configuración común consiste en anunciar la ruta predeterminada (0.0.0.0/0) a través de la sesión de Protocolo de puerta de enlace de borde (BGP). Esto obliga a que el tráfico que sale de la red virtual se reenvíe a la red local del cliente que puede quitar el tráfico, lo que interrumpirá los flujos de salida. Para superar este comportamiento predeterminado, se puede configurar [Ruta definida por el usuario (UDR)](/azure/virtual-network/virtual-networks-udr-overview#user-defined) (0.0.0.0/0) y el próximo salto será *Internet*. Como UDR tiene prioridad sobre BGP, el tráfico se destinará a Internet.

## <a name="securing-outbound-traffic-with-firewall"></a>Protección del tráfico de salida con el firewall

Si quiere proteger el tráfico de salida mediante [Azure Firewall](/azure/firewall/overview) o cualquier aplicación virtual para limitar los nombres de dominio, se deben permitir los siguientes nombres de dominio completos (FQDN) en el firewall.

```
prod.warmpath.msftcloudes.com:443
production.diagnostics.monitoring.core.windows.net:443
graph.windows.net:443
*.update.microsoft.com:443
shavamanifestcdnprod1.azureedge.net:443
login.live.com:443
wdcp.microsoft.com:443
login.microsoftonline.com:443
azureprofilerfrontdoor.cloudapp.net:443
*.core.windows.net:443
*.servicebus.windows.net:443
shoebox2.metrics.nsatc.net:443
prod-dsts.dsts.core.windows.net:443
*.identity.azure.net:443
*.vault.azure.net:443
ocsp.msocsp.com:80
*.windowsupdate.com:80
ocsp.digicert.com:80
go.microsoft.com:80
dmd.metaservices.microsoft.com:80
www.msftconnecttest.com:80
crl.microsoft.com:80
www.microsoft.com:80
adl.windows.com:80
crl3.digicert.com:80
```

También debe definir la [tabla de enrutamiento](/azure/virtual-network/virtual-networks-udr-overview) en la subred con las [direcciones de administración](#azure-data-explorer-management-ip-addresses) y las [direcciones de supervisión de estado](#health-monitoring-addresses) con el próximo salto *Internet* para evitar problemas de rutas asimétricas.

Por ejemplo, para la región **Oeste de EE. UU.** , se deben definir los UDR siguientes:

| Nombre | Prefijo de dirección | Próximo salto |
| --- | --- | --- |
| ADX_Management | 13.64.38.225/32 | Internet |
| ADX_Monitoring | 23.99.5.162/32 | Internet |

## <a name="deploy-azure-data-explorer-cluster-into-your-vnet-using-an-azure-resource-manager-template"></a>Implementación de un clúster de Azure Data Explorer en la red virtual mediante una plantilla de Azure Resource Manager

Para implementar el clúster de Azure Data Explorer en la red virtual, use la plantilla de Azure Resource Manager [Implementación del clúster de Azure Data Explorer en la red virtual](https://azure.microsoft.com/resources/templates/101-kusto-vnet/).

Esta plantilla crea el clúster, la red virtual, la subred, el grupo de seguridad de red y las direcciones IP públicas.
