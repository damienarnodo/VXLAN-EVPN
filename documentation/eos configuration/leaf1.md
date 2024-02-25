# Leaf 1 configuration

## Pre-config Setup

Enable the multi-agent configuration on Arista switch to be able to use EVPN feature

```config
service routing protocols model multi-agent
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
leaf1# show bgp summary
```
