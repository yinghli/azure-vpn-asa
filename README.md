# Azure IPSec VPN with Cisco ASA using BGP
Setup VPN between Azure and Cisco ASA with BGP

Cisco ASA software version 9.8 support Virtual Tunnel Interface (VTI) with BGP (static VTI). <br>
https://www.cisco.com/c/en/us/td/docs/security/asa/asa98/release/notes/asarn98.html#reference_s3l_4v2_gy <br>
This feature allow setup BGP neighbor on top of IPSec tunnel with IKEv2. <br>
This documentation will desctrip how to setup IPSec VPN with Azure VPN gateway using BGP. <br>

# Azure VPN Setup 
In Azure side, we will use Azure Portal to setup all vpn steps. Powershell and Azure CLI can do the same setup. <br>
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
Azure BGP Public IP   | 139.219.100.216
Azure BGP peer IP     | 10.10.1.254
VPN Local Gateway Name| VPNLocalGW
On Prem Public IP     | 123.121.211.229
On Prem BGP ASN       | 65510
On Prem BGP Peer IP   | 192.168.2.1
IPSec Pre-share Key   | Microsoft123!

* Setup VNET in Azure <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/VNET.PNG) 
* Setup Gateway Subnet <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GWsubnet-1.png)
* Add Gateway Subnet <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GWSubnet1-1.PNG)
* Setup VPN Gateway <br>
Setup VPN Gateway will use 45 minutes. <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GW.PNG)
* Check VPN Gateway Status<br>
After the VPN setup, you can check public IP address for IPSec VPN setup. <br>
139.219.100.216 is Azure VPN gateway public IP address. <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GWOverview1.PNG)
* Check VPN Gateway BGP Information<br>
Check VPN gateway configuraion, you will get Azure side BGP ASN and BGP peer information.<br>
65500 is Azure VPN gateway BGP AS number. <br>
10.10.1.254 is Azure VPN gateway BGP peer IP address. <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GWStatus1.png)
* Setup Local Network Gateway <br>
Local gateway represent customer on prem ASA setup. <br>
65510 is customer ASA BGP AS number. <br>
123.121.211.229 is customer ASA public IP address. <br>
192.168.2.1 is cusotmer ASA BGP peer IP address, this is VTI address. <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/LocalGW1.png)
* Setup Connection <br>
Setup IPSec VPN on Azure site, pre share key password must be same as customer on prem ASA. <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/Connection.PNG)
* Enable Connection BGP <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/ConnectionBGP.PNG)

# Cisco ASA Setup
In Cisco ASA side, we will use CLI setup all vpn steps. <br>
We will use below parameters to setup. <br>

Parameters            | Values
----------------------| -------------
IKEv2 policy          | 1
IKEv2 encryption      | aes-256 aes-192 aes
IKEv2 integrity       | sha256 sha
DH group              | 2
PRF                   | sha
IKEv2 IPSec proposal  | SET1
IPSec protocol        | ESP
IPSec encryption      | aes-256 aes-192 aes
IPSec integrity       | sha-256
IPSec profile         | PROFILE1
ASA Public IP         | 123.121.211.229
ASA BGP ASN           | 65510
ASA BGP Peer IP       | 192.168.2.1
IPSec Pre-share Key   | Microsoft123!

* Setup ASA Interface <br>
```
interface GigabitEthernet0/0
 nameif outside
 security-level 0
 ip address 123.121.211.229 255.255.255.0
!
interface GigabitEthernet0/1
 nameif inside
 security-level 100
 ip address 192.168.0.1 255.255.255.0
!
```
* Setup IKEv2 Profile <br>
```
crypto ikev2 policy 1
 encryption aes-256 aes-192 aes
 integrity sha256 sha
 group 2
 prf sha
 lifetime seconds 86400
crypto ikev2 enable outside
```
* Setup IPSec Profile <br>
```
crypto ipsec ikev2 ipsec-proposal SET1
 protocol esp encryption aes-256 aes-192 aes
 protocol esp integrity sha-256
crypto ipsec profile PROFILE1
 set ikev2 ipsec-proposal SET1
```
* Setup IPSec pre-share Key <br?
```
tunnel-group 139.219.100.216 type ipsec-l2l
tunnel-group 139.219.100.216 ipsec-attributes
 ikev2 remote-authentication pre-shared-key Microsoft123!
 ikev2 local-authentication pre-shared-key Microsoft123!
```
* Setup VTI <br>
```
interface Tunnel1
 nameif vti
 ip address 192.168.2.1 255.255.255.0
 tunnel source interface outside
 tunnel destination 139.219.100.216
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile PROFILE1
```
* Setup Route <br>
Setup default route to "outside" interface. <br>
Setup Azure BGP peer traffic to "VTI" interface.
```
route outside 0.0.0.0 0.0.0.0 123.121.211.1 1
route vti 10.10.1.0 255.255.255.0 10.10.1.254 1
```
* Setup BGP <br>
```
router bgp 65510
 bgp log-neighbor-changes
 address-family ipv4 unicast
  neighbor 10.10.1.254 remote-as 65500
  neighbor 10.10.1.254 ebgp-multihop 2
  neighbor 10.10.1.254 activate
  network 192.168.0.0
  no auto-summary
  no synchronization
 exit-address-family
```

# Verify IPSec VPN and BGP
* Azure VPN Status
* Azure BGP Status
* ASA VPN Status
* ASA BGP Status
