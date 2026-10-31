# VLANs and addressing

Inter-VLAN routing is done with **router-on-a-stick** on R1 (`GigabitEthernet0/1` with 802.1Q subinterfaces).

| VLAN | Name | Subnet | Gateway (R1) | Addressing | Purpose |
|---|---|---|---|---|---|
| 10 | PCs | 192.168.10.0/24 | 192.168.10.1 (Gi0/1.10) | DHCP (.1–.20 excluded) | Administrative equipment and workstations |
| 20 | WIFI | 192.168.20.0/24 | 192.168.20.1 (Gi0/1.20) | DHCP (.1–.20 excluded) | Client devices connected through wireless access points |
| 30 | IOT | 192.168.30.0/24 | 192.168.30.1 (Gi0/1.30) | DHCP (.1–.20 excluded) | IoT devices and data-acquisition equipment |
| 40 | MQTT | 192.168.40.0/24 | 192.168.40.1 (Gi0/1.40) | Static (no DHCP pool) | Infrastructure services, incl. the MQTT broker (192.168.40.10) |

DHCP pools `VLAN10`, `VLAN20`, `VLAN30` live on R1 and hand out the gateway and DNS `8.8.8.8`.

## Router-to-router link

| Device | Interface | IP |
|---|---|---|
| R1 | Gi0/0 | 10.0.0.1/30 |
| R2 | Gi0/0 | 10.0.0.2/30 |

## Routing (static)

- **R1:** default route `0.0.0.0/0` via `10.0.0.2` (R2).
- **R2:** one static route per LAN subnet (`192.168.10/20/30/40.0/24`) via `10.0.0.1` (R1).

## Key hosts

| Device | IP | VLAN |
|---|---|---|
| MQTT Broker (Server-PT) | 192.168.40.10 | 40 |
| SBC2 (publisher, Wi-Fi via AP3-EXTERIOR) | DHCP | 20 |
| Laptop1 – Sala Técnica (subscriber) | 192.168.10.23 (DHCP) | 10 |

The publisher (VLAN 20), the subscriber (VLAN 10) and the broker (VLAN 40) are in different VLANs: MQTT traffic between them is routed by R1 (router-on-a-stick).

Device configurations: [`configs/R1.txt`](../configs/R1.txt), [`configs/R2.txt`](../configs/R2.txt).

## Justification of VLAN segmentation

SBC2 connects wirelessly through AP3-EXTERIOR and gets its address from the VLAN 20 DHCP pool. This lets the device communicate with the other hosts of its segment. SBC2 acts as an intermediary with a client-type connection (the camera is attached to its IoT port), so it sits in the client VLAN, while VLAN 30 is intended for the other IoT devices and VLAN 40 hosts infrastructure services, including the MQTT broker.
