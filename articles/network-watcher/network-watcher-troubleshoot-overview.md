---
title: Introduction to resource troubleshooting
titleSuffix: Azure Network Watcher
description: This page provides an overview of the Network Watcher resource troubleshooting capabilities
services: network-watcher
documentationcenter: na
author: halkazwini
ms.service: network-watcher
ms.topic: troubleshooting
ms.tgt_pltfrm: na
ms.workload:  infrastructure-services
ms.date: 03/31/2022
ms.author: halkazwini
ms.custom: engagement-fy23
---

# Introduction to resource troubleshooting in Azure Network Watcher

Virtual Network Gateways provide connectivity between on-premises resources and other virtual networks within Azure. Monitoring gateways and their connections are critical to ensuring communication is not broken. Network Watcher provides the capability to troubleshoot gateways and connections. The capability can be called through the portal, PowerShell, Azure CLI, or REST API. When called, Network Watcher diagnoses the health of the gateway, or connection, and returns the appropriate results. The request is a long running transaction. The results are returned once the diagnosis is complete.

![Screenshot shows Network Watcher V P N Diagnostics.][2]

## Results

The preliminary results returned give an overall picture of the health of the resource. Deeper information can be provided for resources as shown in the following section:

The following list is the values returned with the troubleshoot API:

* **startTime** - This value is the time the troubleshoot API call started.
* **endTime** - This value is the time when the troubleshooting ended.
* **code** - This value is UnHealthy, if there is a single diagnosis failure.
* **results** - Results is a collection of results returned on the Connection or the virtual network gateway.
    * **id** - This value is the fault type.
    * **summary** - This value is a summary of the fault.
    * **detailed** - This value provides a detailed description of the fault.
    * **recommendedActions** - This property is a collection of recommended actions to take.
      * **actionText** - This value contains the text describing what action to take.
      * **actionUri** - This value provides the URI to documentation on how to act.
      * **actionUriText** - This value is a short description of the action text.

The following tables show the different fault types (id under results from the preceding list) that are available and if the fault creates logs.

### Gateway

| Fault Type | Reason | Log|
|---|---|---|
| NoFault | When no error is detected |Yes|
| GatewayNotFound | Cannot find gateway or gateway is not provisioned |No|
| PlannedMaintenance |  Gateway instance is under maintenance  |No|
| UserDrivenUpdate | This fault occurs when a user update is in progress. The update could be a resize operation. | No |
| VipUnResponsive | This fault occurs when the primary instance of the gateway can't be reached due to a health probe failure. | No |
| PlatformInActive | There is an issue with the platform. | No|
| ServiceNotRunning | The underlying service is not running. | No|
| NoConnectionsFoundForGateway | No connections exist on the gateway. This fault is only a warning.| No|
| ConnectionsNotConnected | Connections are not connected. This fault is only a warning.| Yes|
| GatewayCPUUsageExceeded | The current gateway CPU usage is > 95%. | Yes |

### Connection

| Fault Type | Reason | Log|
|---|---|---|
| NoFault | When no error is detected |Yes|
| GatewayNotFound | Cannot find gateway or gateway is not provisioned |No|
| PlannedMaintenance | Gateway instance is under maintenance  |No|
| UserDrivenUpdate | This fault occurs when a user update is in progress. The update could be a resize operation.  | No |
| VipUnResponsive | This fault occurs when the primary instance of the gateway can't be reached due to a health probe failure. | No |
| ConnectionEntityNotFound | Connection configuration is missing | No |
| ConnectionIsMarkedDisconnected | The connection is marked "disconnected" |No|
| ConnectionNotConfiguredOnGateway | The underlying service does not have the connection configured. | Yes |
| ConnectionMarkedStandby | The underlying service is marked as standby.| Yes|
| Authentication | Preshared key mismatch | Yes|
| PeerReachability | The peer gateway is not reachable. | Yes|
| IkePolicyMismatch | The peer gateway has IKE policies that are not supported by Azure. | Yes|
| WfpParse Error | An error occurred parsing the WFP log. |Yes|

## Supported Gateway types

The following table lists which gateways and connections are supported with Network Watcher troubleshooting:

| Gateway or connection | Supported  |
|---------|---------|
|**Gateway types**   |         |
|VPN      | Supported        |
|ExpressRoute | Not Supported |
|**VPN types** | |
|Route Based | Supported|
|Policy Based | Not Supported|
|**Connection types**||
|IPSec| Supported|
|VNet2Vnet| Supported|
|ExpressRoute| Not Supported|
|VPNClient| Not Supported|

## Log files

The resource troubleshooting log files are stored in a storage account after resource troubleshooting is finished. The following image shows the example contents of a call that resulted in an error.

![zip file][1]

> [!NOTE]
> 1. In some cases, only a subset of the logs files is written to storage.
> 2. For newer Gateway versions, the IkeErrors.txt, Scrubbed-wfpdiag.txt and wfpdiag.txt.sum have been replaced by an IkeLogs.txt file that contains the whole IKE activity (not just errors).

For instructions on downloading files from Azure storage accounts, refer to [Get started with Azure Blob storage using .NET](../storage/blobs/storage-quickstart-blobs-dotnet.md). Another tool that can be used is Storage Explorer. More information about Storage Explorer can be found here at the following link: [Storage Explorer](https://storageexplorer.com/)

### ConnectionStats.txt

The **ConnectionStats.txt** file contains overall stats of the Connection, including ingress and egress bytes, Connection status, and the time the Connection was established.

> [!NOTE]
> If the call to the troubleshooting API returns healthy, the only thing returned in the zip file is a **ConnectionStats.txt** file.

The contents of this file are similar to the following example:

```
Connectivity State : Connected
Remote Tunnel Endpoint :
Ingress Bytes (since last connected) : 288 B
Egress Bytes (Since last connected) : 288 B
Connected Since : 2/1/2017 8:22:06 PM
```

### CPUStats.txt

The **CPUStats.txt** file contains CPU usage and memory available at the time of testing.  The contents of this file is similar to the following example:

```
Current CPU Usage : 0 % Current Memory Available : 641 MBs
```

### IKElogs.txt

The **IKElogs.txt** file contains any IKE activity that was found during monitoring.

The following example shows the contents of an IKElogs.txt file.

```
Remote <IPaddress>:500: Local <IPaddress>:500: [RECEIVED][SA_AUTH] Received IKE AUTH message
Remote <IPaddress>:500: Local <IPaddress>:500: Received Traffic Selector payload request- [Tsid 0x729  ]Number of TSIs 2: StartAddress 10.20.0.0 EndAddress 10.20.255.255 PortStart 0 PortEnd 65535 Protocol 0, StartAddress 192.168.100.0 EndAddress 192.168.100.255 PortStart 0 PortEnd 65535 Protocol 0 Number of TSRs 1:StartAddress 0.0.0.0 EndAddress 255.255.255.255 PortStart 0 PortEnd 65535 Protocol 0
Remote <IPaddress>:500: Local <IPaddress>:500: [SEND] Proposed Traffic Selector payload will be (Final Negotiated) - [Tsid 0x729  ]Number of TSIs 2: StartAddress 10.20.0.0 EndAddress 10.20.255.255 PortStart 0 PortEnd 65535 Protocol 0, StartAddress 192.168.100.0 EndAddress 192.168.100.255 PortStart 0 PortEnd 65535 Protocol 0 Number of TSRs 1:StartAddress 0.0.0.0 EndAddress 255.255.255.255 PortStart 0 PortEnd 65535 Protocol 0
Remote <IPaddress>:500: Local <IPaddress>:500: [RECEIVED]Received IPSec payload: Policy1:Cipher=DESIntegrity=Md5
IkeCleanupQMNegotiation called with error 13868 and flags a
Remote <IPaddress>:500: Local <IPaddress>:500: [SEND][NOTIFY] Sending Notify Message - Policy Mismatch
```

### IKEErrors.txt

The **IKEErrors.txt** file contains any IKE errors that were found during monitoring.

The following example shows the contents of an IKEErrors.txt file. Your errors may be different depending on the issue.

```
Error: Authentication failed. Check shared key. Check crypto. Check lifetimes. 
	 based on log : Peer failed with Windows error 13801(ERROR_IPSEC_IKE_AUTH_FAIL)
Error: On-prem device sent invalid payload. 
	 based on log : IkeFindPayloadInPacket failed with Windows error 13843(ERROR_IPSEC_IKE_INVALID_PAYLOAD)
```

### Scrubbed-wfpdiag.txt

The **Scrubbed-wfpdiag.txt** log file contains the wfp log. This log contains logging of packet drop and IKE/AuthIP failures.

The following example shows the contents of the Scrubbed-wfpdiag.txt file. In this example, the shared key of a Connection was not correct as can be seen from the third line from the bottom. The following example is just a snippet of the entire log, as the log can be lengthy depending on the issue.

```
...
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|Deleted ICookie from the high priority thread pool list
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|IKE diagnostic event:
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|Event Header:
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Timestamp: 1601-01-01T00:00:00.000Z
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Flags: 0x00000106
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|    Local address field set
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|    Remote address field set
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|    IP version field set
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  IP version: IPv4
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  IP protocol: 0
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Local address: 13.78.238.92
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Remote address: 52.161.24.36
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Local Port: 0
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Remote Port: 0
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Application ID:
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  User SID: <invalid>
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|Failure type: IKE/Authip Main Mode Failure
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|Type specific info:
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Failure error code:0x000035e9
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|    IKE authentication credentials are unacceptable
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|
[0]0368.03A4::02/02/2017-17:36:01.496 [ikeext] 3038|52.161.24.36|  Failure point: Remote
...
```

### wfpdiag.txt.sum

The **wfpdiag.txt.sum** file is a log showing the buffers and events processed.

The following example is the contents of a wfpdiag.txt.sum file.
```
Files Processed:
	C:\Resources\directory\924336c47dd045d5a246c349b8ae57f2.GatewayTenantWorker.DiagnosticsStorage\2017-02-02T17-34-23\wfpdiag.etl
Total Buffers Processed 8
Total Events  Processed 2169
Total Events  Lost      0
Total Format  Errors    0
Total Formats Unknown   486
Elapsed Time            330 sec
+-----------------------------------------------------------------------------------+
|EventCount    EventName            EventType   TMF                                 |
+-----------------------------------------------------------------------------------+
|        36    ikeext               ike_addr_utils_c844  a0c064ca-d954-350a-8b2f-1a7464eef8b6|
|        12    ikeext               ike_addr_utils_c857  a0c064ca-d954-350a-8b2f-1a7464eef8b6|
|        96    ikeext               ike_addr_utils_c832  a0c064ca-d954-350a-8b2f-1a7464eef8b6|
|         6    ikeext               ike_bfe_callbacks_c133  1dc2d67f-8381-6303-e314-6c1452eeb529|
|         6    ikeext               ike_bfe_callbacks_c61  1dc2d67f-8381-6303-e314-6c1452eeb529|
|        12    ikeext               ike_sa_management_c5698  7857a320-42ee-6e90-d5d9-3f414e3ea2d3|
|         6    ikeext               ike_sa_management_c8447  7857a320-42ee-6e90-d5d9-3f414e3ea2d3|
|        12    ikeext               ike_sa_management_c494  7857a320-42ee-6e90-d5d9-3f414e3ea2d3|
|        12    ikeext               ike_sa_management_c642  7857a320-42ee-6e90-d5d9-3f414e3ea2d3|
|         6    ikeext               ike_sa_management_c3162  7857a320-42ee-6e90-d5d9-3f414e3ea2d3|
|        12    ikeext               ike_sa_management_c3307  7857a320-42ee-6e90-d5d9-3f414e3ea2d3|
```

## Considerations 
* Only one troubleshoot operation can be run at a time per subscription. To run another troubleshoot operation, wait for the previous one to complete. Triggering more operations while a previous one hasn't completed will cause subsequent operations to fail. 
* CLI Bug: If you are using Azure CLI to run the command, the VPN Gateway and the Storage account need to be in same resource group. Customers with the resources in different resource groups can use PowerShell or the Azure portal instead.  


## Next steps

To learn how to diagnose a problem with a gateway or gateway connection, see [Diagnose communication problems between networks](diagnose-communication-problem-between-networks.md).
<!--Image references-->

[1]: ./media/network-watcher-troubleshoot-overview/gateway-tenant-worker-logs-new.png
[2]: ./media/network-watcher-troubleshoot-overview/portal.png
