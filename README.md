Azure IPSec VPN with Cisco ASA using BGP
==========================================
Cisco ASA software version 9.8 support Virtual Tunnel Interface (VTI) with BGP (static VTI). <br>
You can check the [release notes](https://www.cisco.com/c/en/us/td/docs/security/asa/asa98/release/notes/asarn98.html#reference_s3l_4v2_gy) <br>
This feature allows setup BGP neighbor on top of IPSec tunnel with IKEv2. <br>
This documentation will describe how to setup IPSec VPN with Azure VPN gateway using BGP. <br>

Topology
----------------------------------------
![](https://github.com/yinghli/azure-vpn-asa/blob/master/ASA9.8.png)

Azure VPN Setup 
----------------------------------------
In Azure side, we will use Azure Portal to setup all vpn configuration. PowerShell and Azure CLI can do the same setup. <br>
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
VPN Type              | Route-based
VPN SKU               | VpnGw1
Azure BGP ASN         | 65500
Azure BGP Public IP   | 139.219.100.216
Azure BGP peer IP     | 10.10.1.254
VPN Local Gateway Name| VPNLocalGW
On Premise Public IP  | 123.121.211.229
On Premise BGP ASN    | 65510
On Premise BGP Peer IP| 192.168.2.1
IPSec Pre-share Key   | Microsoft123!

> **Note:** In IKEv2 and IPSec parameters setup, we will use Azure default values. If you want to setup customized values, please check [here](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-ipsecikepolicy-rm-powershell) <br>
> **Note:** Azure VPN gateway cryptographic can be found [here](https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-compliance-crypto)

## Setup VNET in Azure <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/VNET.PNG) 
## Setup Gateway Subnet <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GWsubnet-1.png)
## Add Gateway Subnet <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GWSubnet1-1.PNG)
## Setup VPN Gateway <br>
Setup VPN Gateway will use 45 minutes. <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GW.PNG)
## Check VPN Gateway Status<br>
After the VPN setup, you can check public IP address for IPSec VPN setup. <br>
139.219.100.216 is Azure VPN gateway public IP address. <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GWOverview1.PNG)
## Check VPN Gateway BGP Information<br>
Check VPN gateway configuration, you will get Azure side BGP ASN and BGP peer information.<br>
65500 is Azure VPN gateway BGP AS number. <br>
10.10.1.254 is Azure VPN gateway BGP peer IP address. <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/GWStatus1.png)
## Setup Local Network Gateway <br>
Local gateway represent customer on prem ASA setup. <br>
65510 is customer ASA BGP AS number. <br>
123.121.211.229 is customer ASA public IP address. <br>
192.168.2.1 is customer ASA BGP peer IP address, this is VTI address. <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/LocalGW1.png)
## Setup Connection <br>
Setup IPSec VPN on Azure site, pre-share key password must be same as customer on premise ASA. <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/Connection.PNG)
## Enable Connection BGP <br>
![](https://github.com/yinghli/azure-vpn-asa/blob/master/ConnectionBGP.PNG)

Cisco ASA Setup
----------------------------------------
In Cisco ASA side, we will use CLI setup all vpn configuration. <br>
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

## Setup ASA Interface <br>
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
## Setup IKEv2 Profile <br>
```
crypto ikev2 policy 1
 encryption aes-256 aes-192 aes
 integrity sha256 sha
 group 2
 prf sha
 lifetime seconds 86400
crypto ikev2 enable outside
```
## Setup IPSec Profile <br>
```
crypto ipsec ikev2 ipsec-proposal SET1
 protocol esp encryption aes-256 aes-192 aes
 protocol esp integrity sha-256
crypto ipsec profile PROFILE1
 set ikev2 ipsec-proposal SET1
```
## Setup IPSec pre-share Key <br>
```
tunnel-group 139.219.100.216 type ipsec-l2l
tunnel-group 139.219.100.216 ipsec-attributes
 ikev2 remote-authentication pre-shared-key Microsoft123!
 ikev2 local-authentication pre-shared-key Microsoft123!
```
## Setup VTI <br>
```
interface Tunnel1
 nameif vti
 ip address 192.168.2.1 255.255.255.0
 tunnel source interface outside
 tunnel destination 139.219.100.216
 tunnel mode ipsec ipv4
 tunnel protection ipsec profile PROFILE1
```
## Setup Route <br>
Setup default route to "outside" interface. <br>
Setup Azure BGP peer traffic to "VTI" interface.
```
route outside 0.0.0.0 0.0.0.0 123.121.211.1 1
route vti 10.10.1.0 255.255.255.0 10.10.1.254 1
```
## Setup BGP <br>
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

Verify IPSec VPN and BGP
----------------------------------------
## Azure VPN Status <br>
Powershell command **Get-AzureRmVirtualNetworkGatewayConnection -Name ASA -ResourceGroupName VPN** can check VPN status.<br>
You can see the ConnectionStatus is **Connected**<br>
```
PS C:\WINDOWS\system32> Get-AzureRmVirtualNetworkGatewayConnection -Name ASA -ResourceGroupName VPN

Name                    : ASA
ResourceGroupName       : VPN
Location                : chinanorth
ProvisioningState       : Succeeded
Tags                    : 
AuthorizationKey        : 
VirtualNetworkGateway1  : "/subscriptions/1ce3bd2d-3193-4af3-8d2a-9b9ef3458277/resourceGroups/VPN/providers/Microsoft.Network/virtualNetworkGateways/VP
                          NGW"
VirtualNetworkGateway2  : 
LocalNetworkGateway2    : "/subscriptions/1ce3bd2d-3193-4af3-8d2a-9b9ef3458277/resourceGroups/VPN/providers/Microsoft.Network/localNetworkGateways/VPNL
                          ocalGW"
Peer                    : 
RoutingWeight           : 0
SharedKey               : Microsoft123!
ConnectionStatus        : Connected
EgressBytesTransferred  : 25054
IngressBytesTransferred : 17388
TunnelConnectionStatus  : []
```
## Azure BGP Status
Powershell command **Get-AzureRmVirtualNetworkGatewayBgpPeerStatus -VirtualNetworkGatewayName VPNGW -ResourceGroupName VPN** can check BGP State. <br>
From the output, BGP State is **Connected**. <br>
```
PS C:\WINDOWS\system32> Get-AzureRmVirtualNetworkGatewayBgpPeerStatus -VirtualNetworkGatewayName VPNGW -ResourceGroupName VPN 

Asn               : 65510
ConnectedDuration : 00:09:52.5820139
LocalAddress      : 10.10.1.254
MessagesReceived  : 13
MessagesSent      : 14
Neighbor          : 192.168.2.1
RoutesReceived    : 1
State             : Connected
```
## Azure BGP Route Learned from ASA
Powershell command **Get-AzureRmVirtualNetworkGatewayLearnedRoute -VirtualNetworkGatewayName VPNGW -ResourceGroupName VPN** can check BGP learned route from ASA. <br>
```
PS C:\WINDOWS\system32> Get-AzureRmVirtualNetworkGatewayLearnedRoute -VirtualNetworkGatewayName VPNGW -ResourceGroupName VPN


AsPath       : 
LocalAddress : 10.10.1.254
Network      : 10.10.0.0/23
NextHop      : 
Origin       : Network
SourcePeer   : 10.10.1.254
Weight       : 32768

AsPath       : 
LocalAddress : 10.10.1.254
Network      : 192.168.2.1/32
NextHop      : 
Origin       : Network
SourcePeer   : 10.10.1.254
Weight       : 32768

AsPath       : 65510
LocalAddress : 10.10.1.254
Network      : 192.168.0.0/24
NextHop      : 192.168.2.1
Origin       : EBgp
SourcePeer   : 192.168.2.1
Weight       : 32768
```
## ASA IKEv2 Status <br>
ASA CLI command **show crypto ikev2 sa** can check the IKEv2 status. <br>
From the output, you can see Status is **UP-ACTIVE**. <br>
```
ciscoasa# show crypto ikev2 sa

IKEv2 SAs:

Session-id:54426, Status:UP-ACTIVE, IKE count:1, CHILD count:1

Tunnel-id Local                                               Remote                                                  Status         Role
180382363 123.121.211.229/4500                                 139.219.100.216/4500                                    **READY**    INITIATOR
      Encr: AES-CBC, keysize: 256, Hash: SHA96, DH Grp:2, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/415 sec
Child sa: local selector  0.0.0.0/0 - 255.255.255.255/65535
          remote selector 0.0.0.0/0 - 255.255.255.255/65535
          ESP spi in/out: 0x8d2c8231/0xea8a498e
```
## ASA IPSec Status <br>
Use command **show crypto ipsec sa detail** can check IPSec status. <br>
From the output, IPSec VPN tunnel have encaps and decaps packets. It means IPSec VPN tunnel setup correctly.<br>
Both SPI is **Active** <br>
```
ciscoasa# show crypto ipsec sa detail
interface: vti
    Crypto map tag: __vti-crypto-map-4-0-1, seq num: 65280, local addr: 123.121.211.229

      local ident (addr/mask/prot/port): (0.0.0.0/0.0.0.0/0/0)
      remote ident (addr/mask/prot/port): (0.0.0.0/0.0.0.0/0/0)
      current_peer: 139.219.100.216
      
      #pkts encaps: 37, #pkts encrypt: 37, #pkts digest: 37
      #pkts decaps: 93, #pkts decrypt: 93, #pkts verify: 93
      local crypto endpt.: 123.121.211.229/4500, remote crypto endpt.: 139.219.100.216/4500
      path mtu 1500, ipsec overhead 86(52), media mtu 1500
      PMTU time remaining (sec): 0, DF policy: copy-df
      ICMP error validation: disabled, TFC packets: disabled
      current outbound spi: EA8A498E
      current inbound spi : 8D2C8231

    inbound esp sas:
      spi: 0x8D2C8231 (2368504369)
         SA State: active
         transform: esp-aes-256 esp-sha-256-hmac no compression
         in use settings ={L2L, Tunnel,  NAT-T-Encaps, IKEv2, VTI, }
         slot: 0, conn_id: 222928896, crypto-map: __vti-crypto-map-4-0-1
         sa timing: remaining key lifetime (kB/sec): (4055035/28244)
         IV size: 16 bytes
         replay detection support: Y
         Anti replay bitmap:
          0xFFFFFFFF 0xFFFFFFFF
    outbound esp sas:
      spi: 0xEA8A498E (3934931342)
         SA State: active
         transform: esp-aes-256 esp-sha-256-hmac no compression
         in use settings ={L2L, Tunnel,  NAT-T-Encaps, IKEv2, VTI, }
         slot: 0, conn_id: 222928896, crypto-map: __vti-crypto-map-4-0-1
         sa timing: remaining key lifetime (kB/sec): (4008958/28242)
         IV size: 16 bytes
         replay detection support: Y
         Anti replay bitmap:
          0x00000000 0x00000001

```
## ASA BGP Status <br>
Command **show bgp neighbors** can check ASA BGP status. <br>
From the output, BGP neighbors is **Established**.<br>
```
ciscoasa# show bgp neighbors

BGP neighbor is 10.10.1.254,  context single_vf,  remote AS 65500, external link
  BGP version 4, remote router ID 10.10.1.254
  BGP state = Established, up for 00:04:14
  Last read 00:00:46, last write 00:00:54, hold time is 180, keepalive interval is 60 seconds
  Neighbor sessions:
    1 active, is not multisession capable (disabled)
```
* ASA BGP Route <br>
Command **show bgp** will display the BGP route. <br>
From the output, we can see ASA learn Azure network **10.10.0.0/23** from 10.10.1.254<br>

```
ciscoasa# show bgp

BGP table version is 5, local router ID is 192.168.2.1
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath
Origin codes: i - IGP, e - EGP, ? - incomplete

   Network          Next Hop        Metric LocPrf Weight  Path
*> 10.10.0.0/23     10.10.1.254                        0  65500 i
*> 192.168.0.0      0.0.0.0              0         32768  i
r> 192.168.2.1/32   10.10.1.254                        0  65500 i
```
## ASA Route Table <br>
Command **show route** will display the ASA route table. <br>
From the output, 10.10.0.0/23 already in route table. All traffic go to this subnet will sent to 10.10.1.254.<br>
```
ciscoasa# show route

Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2, V - VPN
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, + - replicated route
Gateway of last resort is 123.121.211.1 to network 0.0.0.0

S*       0.0.0.0 0.0.0.0 [1/0] via 123.121.211.1, outside
B        10.10.0.0 255.255.254.0 [20/0] via 10.10.1.254, 00:04:08
S        10.10.1.0 255.255.255.0 [1/0] via 10.10.1.254, vti
C        192.168.0.0 255.255.255.0 is directly connected, inside
L        192.168.0.1 255.255.255.255 is directly connected, inside
C        192.168.2.0 255.255.255.0 is directly connected, vti
L        192.168.2.1 255.255.255.255 is directly connected, vti
C        123.121.211.0 255.255.255.0 is directly connected, outside
L        123.121.211.229 255.255.255.255 is directly connected, outside
```
