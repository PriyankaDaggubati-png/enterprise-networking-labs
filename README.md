# Network labs in GNS3

These are three labs I built and tested in GNS3. Each lab has a topology, a goal, and full Cisco IOS configs you can paste into your own lab. The configs use Cisco 1900 and 4300 series routers and 2960 and 3850 series switches, the same kind of gear I work with day to day.

## What you need

- GNS3 installed on your machine
- Cisco IOS router and switch images for your GNS3 setup
- The config files in each lab folder below

## Lab 1 - Multi-area OSPF with EIGRP

### Goal

Build a small network with OSPF running in two areas, and connect an EIGRP site through redistribution on R2.

### Topology

```
  192.168.10.0/24              192.168.30.0/24
        |                            |
       R1 ---------- R2 ---------- R3
    Area 0       Area 0 + 1      Area 1
                  |
                  | EIGRP 100
                  |
                 R4
           192.168.40.0/24
```

### Devices and addresses

- R1 - Area 0 router, link to R2 on 10.0.12.1, LAN 192.168.10.1
- R2 - Area border router, Area 0 link 10.0.12.2, Area 1 link 10.0.23.2, EIGRP link 10.0.24.2
- R3 - Area 1 router, link to R2 on 10.0.23.3, LAN 192.168.30.1
- R4 - EIGRP only router, link to R2 on 10.0.24.4, LAN 192.168.40.1

### Config files

- lab-1-ospf-eigrp/r1.txt
- lab-1-ospf-eigrp/r2.txt
- lab-1-ospf-eigrp/r3.txt
- lab-1-ospf-eigrp/r4.txt

### Checks to run

- show ip ospf neighbor on R1, R2, and R3
- show ip route ospf on R3, and look for the EIGRP routes that came through R2
- show ip eigrp neighbors on R2 and R4
- ping 192.168.30.1 from R1, and ping 192.168.40.1 from R3

### What this lab proves

- OSPF neighbors come up across Area 0 and Area 1
- R2 works as an area border router between the two areas
- EIGRP routes reach the OSPF side through redistribution, and OSPF routes reach the EIGRP side the same way

## Lab 2 - Inter-VLAN routing with SVI and Router-on-a-Stick, plus VRRP

### Goal

Route traffic between VLANs two ways, first with a router on a stick, then with SVI interfaces on a multilayer switch, and add VRRP so the gateway stays up if one router fails.

### Topology, Part A - Router-on-a-Stick with VRRP

```
        R1 (VRRP master)      R2 (VRRP backup)
         192.168.10.1          192.168.10.2
         192.168.20.1          192.168.20.2
         192.168.30.1          192.168.30.2
         VIP 192.168.10.254
         VIP 192.168.20.254
         VIP 192.168.30.254
              \                /
               \              /
              SW1 (trunk ports)
              /     |      \
        VLAN 10  VLAN 20  VLAN 30
```

### Topology, Part B - SVI routing on a multilayer switch

```
              SW-CORE
        Vlan10 192.168.10.3
        Vlan20 192.168.20.3
        Vlan30 192.168.30.3
                |
             SW1 (trunk)
           /     |      \
     VLAN 10  VLAN 20  VLAN 30
```

Part A and Part B are two ways to do the same job, so build one at a time.

### Config files

- lab-2-intervlan-vrrp/router-1-roas-vrrp.txt
- lab-2-intervlan-vrrp/router-2-vrrp-backup.txt
- lab-2-intervlan-vrrp/switch-access.txt
- lab-2-intervlan-vrrp/switch-core-svi.txt

### Checks to run

- show ip route on R1, and look for the connected subinterfaces
- show vlan brief on SW1, and check each access port sits in the right VLAN
- show vrrp brief on R1 and R2, and check R1 is master
- Ping from a host in VLAN 10 to a host in VLAN 20
- Shut down the link from R1 to SW1, then ping the VIP again and watch R2 take over as master

### What this lab proves

- Hosts in different VLANs can talk through the router subinterfaces, and through SVI interfaces on the multilayer switch
- VRRP keeps the gateway address alive when the master router goes down
- One gateway address serves the whole VLAN, so hosts never need a config change when the master changes

## Lab 3 - STP, RSTP, and EtherChannel (LACP)

### Goal

Build a triangle of three switches, run rapid STP so one link stays blocked, and bundle the other links into LACP EtherChannels.

### Topology

```
                SW1
            root bridge
           priority 4096
             /       \
      Po1 LACP       Gi0/2
           /           \
         SW2 -- Po2 -- SW3
     priority 8192  priority 12288
```

### Config files

- lab-3-stp-etherchannel/sw1.txt
- lab-3-stp-etherchannel/sw2.txt
- lab-3-stp-etherchannel/sw3.txt

### Checks to run

- show spanning-tree vlan 10 on all three switches, and check SW1 is the root
- show etherchannel summary, and check Po1 and Po2 are in use
- show interfaces status, and find the port STP put into blocking state
- Shut down one member link of Po1, then check the channel stays up on the other member

### What this lab proves

- Rapid STP picks one root bridge and blocks one link so the triangle never loops
- LACP bundles two links into one channel, and the channel survives a single link failure
- The blocked link comes up on its own if the forwarding link fails
