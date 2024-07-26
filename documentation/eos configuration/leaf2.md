# Leaf 2 configuration

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
ip address 10.1.1.3/31
!
interface Ethernet2
description TO_HOST2
switchport mode trunk
!
interface Loopback0
description VTEP
ip address 10.10.110.2/32
!
```

## BGP Protocol

```config
router bgp 65102
router-id 10.10.110.2
maximum-paths 4 ecmp 4
neighbor SPINE_GROUP peer group
neighbor SPINE_GROUP allowas-in 1
neighbor SPINE_GROUP ebgp-multihop 4
neighbor SPINE_GROUP send-community extended
neighbor SPINE_GROUP maximum-routes 12000
neighbor 10.1.1.2 peer group SPINE_GROUP
neighbor 10.1.1.2 remote-as 65001
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
ip address 10.10.110.2/32
!
ip prefix-list VTEP_PREFIX seq 10 permit 10.10.110.2/32
!
route-map RMAP_VTEP permit 10
match ip address prefix-list VTEP_PREFIX
!
router bgp 65102
neighbor 10.10.110.1 peer group VTEP_GROUP
neighbor 10.10.110.1 remote-as 65101
neighbor 10.10.110.1 update-source Loopback0
neighbor VTEP_GROUP peer group
neighbor VTEP_GROUP ebgp-multihop 5
neighbor VTEP_GROUP send-community extended
!
address-family evpn
neighbor 10.10.110.1 activate
!
address-family ipv4
no neighbor 10.10.110.1 activate
redistribute connected route-map RMAP_VTEP
```

To check peer status :

```cli
show bgp summary
show bgp evpn
```
