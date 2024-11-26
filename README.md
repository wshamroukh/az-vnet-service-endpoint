# How to route the traffic through S2S VPN connection to a VNet Service Endpoint enabled Azure service (like storage account)

* This script creates a S2S VPN connection between Azure VPN Gateway and onprem VPN gateway.
* Then a storage account is created, and then VNet Service Endpoint is enabled on the storage account to allow only a specific subnet (vm) in a virtual network (hub1).
* Once the VNet Service Endpoint is enabled on the storage account, the public traffic to the storage account is disabled including the on-premises network and only specific subnets are allowed to access the storage account.
* To allow traffic from on-premise, you must also allow the public IP address from your on-premises or ExressRoute ([check this](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview#secure-azure-service-access-from-on-premises)).
* However in this script, we address the traffic from on-premises network differently:
  * We enable VNet Service Endpoint for storage on GatewaySubnet.
  * On on-premises network, we create a static route to send the traffic originating towards the storage account (which has vnet service endpoint enabled) through the on-premises VPN Gateway
  * On the on-premises VPN Gateway, we create a static route to send the traffic originating towards the storage account via the ipsec tunnel interface.
  * When we try to access the storage account from on-premises network, the traffic will be routed to the on-premises VPN Gateway, which in turn will route the traffic to the Azure VNet VPN Gateway.
  * Then Azure VPN Gateway will route the traffic to the storage account through the optimized route created when we enabled the VNet service endpoint on the Gatewaysubnet.
  
# Note

If P2S VPN is enabled on the gateway and you wanted the P2S VPN clients traffic to the storage account to be sent through the P2S VPN tunnel (instead of going to the internet which is blocked once you enable service endpoint), you could use the command below - in the `custom-routes` parameter add the IP address of the blob endpoint:

`az network vnet-gateway update -g $rg -n $hub1_vnet_name-gw --custom-routes "$blob_ip/32"`

In Azure Powershell us this command - the parameter `CustomRoute` is where you add the ip address of the blob endpoint
```
Set-AzVirtualNetworkGateway
   -VirtualNetworkGateway <PSVirtualNetworkGateway>
   [-GatewaySku <String>]
   [-GatewayDefaultSite <PSLocalNetworkGateway>]
   [-VpnClientAddressPool <String[]>]
   [-VpnClientProtocol <String[]>]
   [-VpnClientRootCertificates <PSVpnClientRootCertificate[]>]
   [-VpnClientRevokedCertificates <PSVpnClientRevokedCertificate[]>]
   [-VpnClientIpsecPolicy <PSIpsecPolicy[]>]
   [-Asn <UInt32>]
   [-PeerWeight <Int32>]
   [-EnableActiveActiveFeature]
   [-DisableActiveActiveFeature]
   [-RemoveAadAuthentication]
   [-CustomRoute <String[]>]
   [-AsJob]
   [-DefaultProfile <IAzureContextContainer>]
   [-WhatIf]
   [-Confirm]
   [<CommonParameters>]

```