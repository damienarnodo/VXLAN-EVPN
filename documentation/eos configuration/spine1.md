# Spine 1 configuration

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
description TO_LEAF01
no switchport
ip address 10.1.1.0/31
!
interface Ethernet2
description TO_LEAF02
no switchport
ip address 10.1.1.2/31
```

## BGP Protocol

```config
router bgp 65001
router-id 10.10.100.1
maximum-paths 4 ecmp 4
neighbor LEAF_GROUP peer group
neighbor LEAF_GROUP allowas-in 1
neighbor LEAF_GROUP ebgp-multihop 4
neighbor LEAF_GROUP send-community extended
neighbor LEAF_GROUP maximum-routes 12000
neighbor 10.1.1.1 peer group LEAF_GROUP
neighbor 10.1.1.1 remote-as 65101
neighbor 10.1.1.3 peer group LEAF_GROUP
neighbor 10.1.1.3 remote-as 65102
!
address-family ipv4
neighbor 10.1.1.1 activate
neighbor 10.1.1.3 activate
```

To check peer status :

```cli
leaf1# show bgp summary
```
