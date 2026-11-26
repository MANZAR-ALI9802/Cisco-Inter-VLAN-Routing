# Inter-VLAN Routing with Router-on-a-Stick (Cisco Packet Tracer)

A Cisco Packet Tracer lab that connects two VLANs, **STAFF** and **STUDENT**, through a router using 802.1Q sub-interfaces. It is the next step after my [VLAN Segmentation & Trunking Lab](https://github.com/MANZAR-ALI9802/VLAN-Segmentation-Trunking-Lab-Cisco-Packet-Tracer-), where VLANs were isolated from each other.

![Topology](images/topology.png)

## Objectives
- Create and name VLANs on a multilayer switch
- Assign access ports to the STAFF and STUDENT VLANs
- Configure a trunk link from the switch to the router
- Configure 802.1Q sub-interfaces on the router, one per VLAN
- Verify that hosts in different VLANs can communicate through the router

## Topology

| Item | Details |
|------|---------|
| Router | Cisco 2621XM (Router0) |
| Switch | Cisco 3560-24PS multilayer switch (Switch3) |
| Router-to-switch link | 802.1Q trunk (FastEthernet 0/0 on the router) |
| Routing method | Router-on-a-stick (sub-interfaces Fa0/0.10 and Fa0/0.20) |

## VLAN and Addressing Plan

| VLAN | Name | Subnet | Default Gateway | Switch Ports | Router Sub-interface |
|------|------|--------|-----------------|--------------|----------------------|
| 10 | STAFF | 10.1.1.0/24 | 10.1.1.254 | Fa0/1-10 | Fa0/0.10 |
| 20 | STUDENT | 20.1.1.0/24 | 20.1.1.254 | Fa0/11-20 | Fa0/0.20 |

| Device | VLAN | IP Address | Subnet Mask | Default Gateway |
|--------|------|------------|-------------|-----------------|
| STAFF-PC2 | 10 | 10.1.1.2 | 255.255.255.0 | 10.1.1.254 |
| STAFF-PC3 | 10 | 10.1.1.3 | 255.255.255.0 | 10.1.1.254 |
| STUDENT-PC2 | 20 | 20.1.1.2 | 255.255.255.0 | 20.1.1.254 |
| STUDENT-PC3 | 20 | 20.1.1.3 | 255.255.255.0 | 20.1.1.254 |

> **Notes on assumptions:** the original diagram labels the port ranges as "F0/0-0/10" and "F0/0-11/20", so this README assumes Fa0/1-10 and Fa0/11-20. The diagram also does not show which switch port connects to the router, so `configs/switch3.txt` uses Fa0/24 as the trunk port. Change these to match your own Packet Tracer file.

## Configuration

Full configs are in [`configs/`](configs/):
- [`router0.txt`](configs/router0.txt)
- [`switch3.txt`](configs/switch3.txt)

**Router sub-interfaces:**

```
interface fastEthernet 0/0
 no shutdown

interface fastEthernet 0/0.10
 encapsulation dot1Q 10
 ip address 10.1.1.254 255.255.255.0

interface fastEthernet 0/0.20
 encapsulation dot1Q 20
 ip address 20.1.1.254 255.255.255.0
```

**Switch VLANs, access ports, and trunk:**

```
vlan 10
 name STAFF
vlan 20
 name STUDENT

interface range fastEthernet 0/1-10
 switchport mode access
 switchport access vlan 10

interface range fastEthernet 0/11-20
 switchport mode access
 switchport access vlan 20

interface fastEthernet 0/24
 switchport trunk encapsulation dot1q
 switchport mode trunk
```

## Verification

On the switch:

```
show vlan brief
show interfaces trunk
```

On the router:

```
show ip interface brief
show ip route
```

`show ip route` should list `10.1.1.0/24` and `20.1.1.0/24` as directly connected through the two sub-interfaces.

Expected ping results:

| Test | Expected Result |
|------|-----------------|
| STAFF-PC2 -> STAFF-PC3 (same VLAN) | Success |
| STUDENT-PC2 -> STUDENT-PC3 (same VLAN) | Success |
| STAFF-PC2 -> 10.1.1.254 (its gateway) | Success |
| STAFF-PC2 -> STUDENT-PC2 (different VLANs, via router) | Success |

The first ping between VLANs can time out while ARP completes. Run it again if that happens.

## How It Works
1. A host in VLAN 10 sends traffic for another subnet to its default gateway, 10.1.1.254.
2. The switch carries the frame to the router over the trunk, tagged with VLAN 10.
3. The router receives it on sub-interface Fa0/0.10, finds the destination on 20.1.1.0/24, and forwards it out sub-interface Fa0/0.20 tagged with VLAN 20.
4. The switch delivers the frame to the destination host in VLAN 20.

## Key Takeaways
- VLANs separate broadcast domains, so a Layer 3 device is needed to move traffic between them.
- One physical router link can serve many VLANs using sub-interfaces.
- Each sub-interface address is the default gateway for its VLAN.
- The `encapsulation dot1Q <vlan-id>` command must match the VLAN ID.
- The switch port toward the router must be a trunk, not an access port.

## Future Work
- [ ] Layer 3 switching with SVIs (`ip routing` on the 3560)
- [ ] DHCP per VLAN
- [ ] Access control lists between STAFF and STUDENT
- [ ] Port security and a restricted trunk allowed-VLAN list

## How to Run
1. Install [Cisco Packet Tracer](https://www.netacad.com/courses/packet-tracer).
2. Open the `.pkt` file in this repository.
3. Verify with the commands above, or paste the configs from `configs/` into each device CLI.

## Tools
Cisco Packet Tracer, Cisco IOS CLI

## Author
Manzar Ali Khan - [LinkedIn](https://www.linkedin.com/in/manzar-ali-902910342) | [GitHub](https://github.com/MANZAR-ALI9802)

## License
MIT
