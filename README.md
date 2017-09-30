# Azure IPSec VPN with Cisco ASA using BGP
Setup VPN between Azure and Cisco ASA with BGP

Cisco ASA software version 9.8 support Virtual Tunnel Interface (VTI) with BGP (static VTI). <br>
https://www.cisco.com/c/en/us/td/docs/security/asa/asa98/release/notes/asarn98.html#reference_s3l_4v2_gy <br>
This feature allow setup BGP neighbor on top of IPSec tunnel with IKEv2. <br>
This documentation will desctrip how to setup IPSec VPN with Azure VPN gateway using BGP. <br>

# Azure VPN Setup 
In this doc, we will use Azure Portal to setup all vpn steps. Powershell and Azure CLI can do the same setup. <br>
* Setup VNET in Azure
![](https://github.com/yinghli/azure-vpn-asa/blob/master/VNET.PNG) 
* Setup Gateway Subnet
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GWsubnet.PNG)
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GWSubnet1.PNG)
* Setup VPN Gateway
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GW.PNG)
* Setup Local Network Gateway
![](https://github.com/yinghli/azure-vpn-asa/blob/master/LocalGW.PNG)
* Setup Connection
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
