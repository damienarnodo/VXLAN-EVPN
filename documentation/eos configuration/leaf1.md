# Leaf 1 configuration

## Pre-config Setup

Disable ZeroTouch provisioning :

```config
en
zerotouch cancel
```

Enable the multi-agent configuration on Arista switch to be able to use EVPN feature

```config
en
conf
service routing protocols model multi-agent
```

Enabling IP Routing

```config
conf
ip routing
```

## IP address Configuration

Configure need to match with [Lab Topology](../../lab_vxlan.yml)

```yml
links:
        - endpoints: ["spine1:eth1", "leaf1:eth1"]
        - endpoints: ["spine1:eth2", "leaf2:eth1"]
        - endpoints: ["leaf1:eth2", "host1:eth1"]
        - endpoints: ["leaf2:eth2", "host2:eth1"]
```

That means :

```config
interface Ethernet1
description TO_SPINE01
no switchport
ip address 10.1.1.1/31
!
interface Ethernet2
description TO_HOST1
switchport mode trunk
!
interface Loopback0
description VTEP
ip address 10.10.110.1/32
!
```

## BGP Protocol

```config
router bgp 65101
router-id 10.10.110.1
maximum-paths 4 ecmp 4
neighbor SPINE_GROUP peer group
neighbor SPINE_GROUP allowas-in 1
neighbor SPINE_GROUP ebgp-multihop 4
neighbor SPINE_GROUP send-community extended
neighbor SPINE_GROUP maximum-routes 12000
neighbor 10.1.1.0 peer group SPINE_GROUP
neighbor 10.1.1.0 remote-as 65001
!
address-family ipv4
neighbor 10.1.1.0 activate
```

To check peer status :

```cli
show bgp summary
```

## BGP EVPN Overlay

```config
interface Loopback0
description VTEP
ip address 10.10.110.1/32
!
ip prefix-list VTEP_PREFIX seq 10 permit 10.10.110.1/32
!
route-map RMAP_VTEP permit 10
match ip address prefix-list VTEP_PREFIX
!
router bgp 65101
neighbor 10.10.110.2 peer group VTEP_GROUP
neighbor 10.10.110.2 remote-as 65102
neighbor 10.10.110.2 update-source Loopback0
neighbor VTEP_GROUP peer group
neighbor VTEP_GROUP ebgp-multihop 5
neighbor VTEP_GROUP send-community extended
!
address-family evpn
neighbor 10.10.110.2 activate
!
address-family ipv4
no neighbor 10.10.110.2 activate
redistribute connected route-map RMAP_VTEP
```

To check peer status :

```cli
show bgp summary
show bgp evpn
```

## VXLAN IRB Configuration

```config
ip virtual-router mac-address 00:0a:bc:10:11:02
!
vlan 50
name IRB_SERVICE
!
interface Vlan50
description CUSTOMER01_ANYCAST_GW
vrf CUSTOMER01
ip address virtual 10.50.0.1/24
!
interface Vxlan1
description VTI
vxlan source-interface Loopback0
vxlan vlan 50 vni 1050
vxlan vlan 50 flood vtep 10.10.110.2
!
router bgp 65101
!
vlan 50
rd 10.50.0.0:65101
route-target both 10.50.0.0:1200
redistribute learned
```

Now, hosts can ping each other, and ping to the VSI work too.

```config
show vxlan address-table
```