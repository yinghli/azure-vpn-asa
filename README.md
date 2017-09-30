# Azure IPSec VPN with Cisco ASA using BGP
Setup VPN between Azure and Cisco ASA with BGP

Cisco ASA software version 9.8 support Virtual Tunnel Interface (VTI) with BGP (static VTI). <br>
https://www.cisco.com/c/en/us/td/docs/security/asa/asa98/release/notes/asarn98.html#reference_s3l_4v2_gy <br>
This feature allow setup BGP neighbor on top of IPSec tunnel with IKEv2. <br>
This documentation will desctrip how to setup IPSec VPN with Azure VPN gateway using BGP. <br>

# Azure VPN Setup 
In this doc, we will use Azure Portal to setup all vpn steps. Powershell and Azure CLI can do the same setup. <br>
We will use below parameters to setup. <br>

Parameters            | Values
----------------------| -------------
VNET Name             | AzureVNET
Address Space         | 10.10.0.0/23
Resource Group        | VPN
Location              | China North
Subnet                | AzureSubnet
Address Range         | 10.10.0.0/24
GatewaySubnet         | 10.10.1.0/24
VPN Gateway Name      | VPNGW
VPN Tyep              | Route-based
VPN SKU               | VpnGw1
Azure BGP ASN         | 65500
Azure BGP Publip IP   | 139.219.100.216
Azure BGP peer IP     | 10.10.1.254
VPN Local Gateway Name| VPNLocalGW
On Prem Public IP     | 123.121.211.229
On Prem BGP ASN       | 65510
On Prem BGP Peer IP   | 192.168.2.1

* Setup VNET in Azure <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/VNET.PNG) 
* Setup Gateway Subnet <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GWsubnet.PNG)
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GWSubnet1.PNG)
* Setup VPN Gateway <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GW.PNG)
* Setup Local Network Gateway <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/LocalGW.PNG)
* Setup Connection <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/Connection.PNG)
![](https://github.com/yinghli/azure-vpn-asa/blob/master/ConnectionBGP.PNG)

# Cisco ASA Setup
* Setup IKEv2 Profile 
* Setup IPSec Profile
* Setup VTI
* Setup BGP

# Verify IPSec VPN and BGP
* Azure VPN Status
* Azure BGP Status
* ASA VPN Status
* ASA BGP Status
