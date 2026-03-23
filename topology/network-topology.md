# ISP Network Deployment — Topology Specification

## Overview

This lab deploys an ISP network in **Cisco Modeling Labs (CML)** using **Nexus 9000v** for core/aggregation and **IOSv** for access devices. The core/aggregation layer runs **EVPN/VXLAN** while the access layer provides **PON** (fiber) and **TWR** (tower/wireless) connectivity.

## Nodes Used in This Lab

### Core — Cisco Nexus 9000v (NX-OS)

| Hostname | Role        | Platform      | Tier |
|----------|-------------|---------------|------|
| core-01  | Core Router | Nexus 9000v   | Core |
| core-02  | Core Router | Nexus 9000v   | Core |

### Aggregation — Cisco Nexus 9000v (NX-OS)

| Hostname | Role        | Platform      | Tier        |
|----------|-------------|---------------|-------------|
| agg-01   | Aggregation | Nexus 9000v   | Aggregation |
| agg-02   | Aggregation | Nexus 9000v   | Aggregation |

### PON Access — Cisco IOSv

| Hostname | Role                    | Platform | Tier   |
|----------|-------------------------|----------|--------|
| pon-01   | Passive Optical Network | IOSv     | Access |
| pon-02   | Passive Optical Network | IOSv     | Access |
| pon-03   | Passive Optical Network | IOSv     | Access |
| pon-04   | Passive Optical Network | IOSv     | Access |

### TWR Access — Cisco IOSv

| Hostname | Role              | Platform | Tier   |
|----------|-------------------|----------|--------|
| twr-01   | Tower Aggregation | IOSv     | Access |
| twr-02   | Tower Aggregation | IOSv     | Access |
| twr-03   | Tower Aggregation | IOSv     | Access |

### Edge — Cisco IOSv

| Hostname | Role        | Platform | Tier |
|----------|-------------|----------|------|
| edge-01  | Edge Router | IOSv     | Edge |
| edge-02  | Edge Router | IOSv     | Edge |

### Test — Cisco IOSv

| Hostname   | Role        | Platform | Tier |
|------------|-------------|----------|------|
| vxlan-test | VXLAN Test  | IOSv     | Test |

**Total: 14 devices**

---

## Topology Structure

```
  Edge           Core            Aggregation            Access
                                                 +-- vxlan-test
                                                 |
  edge-01 --+-- core-01 --+-- agg-01 --+-- twr-01 --+-- twr-03
            |             |            +-- twr-02 ---+
  edge-02 --+-- core-02 --+                (OSPF)
                           |
                           +-- agg-02 --+-- pon-01 --+-- pon-03 -- pon-04
                                        +-- pon-02 --+
                                              (BGP)
  IOSv       Nexus 9000v    Nexus 9000v              IOSv
```

### Connection Summary

- **edge-01, edge-02** — Edge routers facing upstream/peering (IOSv)
- **edge-01, edge-02** connect to **core-01, core-02** via point-to-point links
- **core-01, core-02** connect to **agg-01, agg-02** via point-to-point links
- **vxlan-test** connects to **agg-01** (VXLAN testing)
- **agg-01** connects downstream to **twr-01, twr-02** (tower/wireless, OSPF)
- **twr-03** connects to **twr-01** and **twr-02** (not directly to agg-01)
- **agg-02** connects downstream to **pon-01, pon-02** (fiber/PON, BGP)
- **pon-03** connects to **pon-01** and **pon-02** (not directly to agg-02)
- **pon-04** connects to **pon-03** (not directly to agg-02)
- **Customer cloud (1x)** connects on the access far-right edge

---

## IP Addressing

### Loopback Addresses (100.64.0.x/32)

| Device   | Interface  | Address                   |
|----------|------------|---------------------------|
| core-01  | loopback0  | 100.64.0.1/32 `[VERIFY]`  |
| core-02  | loopback0  | 100.64.0.2/32 `[VERIFY]`  |
| agg-01   | loopback0  | 100.64.0.3/32 `[VERIFY]`  |
| agg-02   | loopback0  | 100.64.0.4/32 `[VERIFY]`  |
| edge-01  | Loopback0  | `[VERIFY]`                |
| edge-02  | Loopback0  | `[VERIFY]`                |
| pon-01   | Loopback0  | `[VERIFY]`                |
| pon-02   | Loopback0  | `[VERIFY]`                |
| pon-03   | Loopback0  | `[VERIFY]`                |
| pon-04   | Loopback0  | `[VERIFY]`                |
| twr-01   | Loopback0  | `[VERIFY]`                |
| twr-02   | Loopback0  | `[VERIFY]`                |
| twr-03     | Loopback0  | `[VERIFY]`                |
| vxlan-test | Loopback0  | `[VERIFY]`                |

> Loopbacks serve as VTEP source addresses for VXLAN tunnels on Nexus 9000v devices.

### Point-to-Point Links (198.51.0.x/31)

| Link                  | Device A | Interface      | IP                        | Device B | Interface      | IP                        |
|-----------------------|----------|----------------|---------------------------|----------|----------------|---------------------------|
| edge-01 ↔ core-01    | edge-01  | Gi0/x          | `[VERIFY]`                | core-01  | Ethernet1/x    | `[VERIFY]`                |
| edge-01 ↔ core-02    | edge-01  | Gi0/x          | `[VERIFY]`                | core-02  | Ethernet1/x    | `[VERIFY]`                |
| edge-02 ↔ core-01    | edge-02  | Gi0/x          | `[VERIFY]`                | core-01  | Ethernet1/x    | `[VERIFY]`                |
| edge-02 ↔ core-02    | edge-02  | Gi0/x          | `[VERIFY]`                | core-02  | Ethernet1/x    | `[VERIFY]`                |
| core-01 ↔ agg-01     | core-01  | Ethernet1/x    | 198.51.0.0/31 `[VERIFY]`  | agg-01   | Ethernet1/x    | 198.51.0.1/31 `[VERIFY]`  |
| core-01 ↔ agg-02     | core-01  | Ethernet1/x    | 198.51.0.2/31 `[VERIFY]`  | agg-02   | Ethernet1/x    | 198.51.0.3/31 `[VERIFY]`  |
| core-02 ↔ agg-01     | core-02  | Ethernet1/x    | 198.51.0.4/31 `[VERIFY]`  | agg-01   | Ethernet1/x    | 198.51.0.5/31 `[VERIFY]`  |
| core-02 ↔ agg-02     | core-02  | Ethernet1/x    | 198.51.0.6/31 `[VERIFY]`  | agg-02   | Ethernet1/x    | 198.51.0.7/31 `[VERIFY]`  |
| agg-01 ↔ vxlan-test  | agg-01   | Ethernet1/x    | `[VERIFY]`                | vxlan-test | Gi0/x        | `[VERIFY]`                |
| agg-01 ↔ twr-01      | agg-01   | Ethernet1/x    | `[VERIFY]`                | twr-01   | Gi0/x          | `[VERIFY]`                |
| agg-01 ↔ twr-02      | agg-01   | Ethernet1/x    | `[VERIFY]`                | twr-02   | Gi0/x          | `[VERIFY]`                |
| twr-01 ↔ twr-03      | twr-01   | Gi0/x          | `[VERIFY]`                | twr-03   | Gi0/x          | `[VERIFY]`                |
| twr-02 ↔ twr-03      | twr-02   | Gi0/x          | `[VERIFY]`                | twr-03   | Gi0/x          | `[VERIFY]`                |
| agg-02 ↔ pon-01      | agg-02   | Ethernet1/x    | `[VERIFY]`                | pon-01   | Gi0/x          | `[VERIFY]`                |
| agg-02 ↔ pon-02      | agg-02   | Ethernet1/x    | `[VERIFY]`                | pon-02   | Gi0/x          | `[VERIFY]`                |
| pon-01 ↔ pon-03      | pon-01   | Gi0/x          | `[VERIFY]`                | pon-03   | Gi0/x          | `[VERIFY]`                |
| pon-02 ↔ pon-03      | pon-02   | Gi0/x          | `[VERIFY]`                | pon-03   | Gi0/x          | `[VERIFY]`                |
| pon-03 ↔ pon-04      | pon-03   | Gi0/x          | `[VERIFY]`                | pon-04   | Gi0/x          | `[VERIFY]`                |

> Interface names: Ethernet for Nexus 9000v, GigabitEthernet (Gi) for IOSv. `[VERIFY]` exact port assignments.

### Customer/Bridge Networks (10.0.0.x)

| Device   | Interface  | Network                 | Description      |
|----------|------------|-------------------------|------------------|
| agg-01   | `[VERIFY]` | 10.0.0.0/24 `[VERIFY]`  | Customer network |
| agg-02   | `[VERIFY]` | 10.0.0.0/24 `[VERIFY]`  | Customer network |

> Exact subnets and VLAN IDs need verification from the diagram.

---

## Underlay Protocol — IS-IS

| IS-IS Area | Scope                       | Color in Diagram    |
|-----------|------------------------------|---------------------|
| 44.0001   | Core/Aggregation `[VERIFY]`  | Green `[VERIFY]`    |
| 50.0001   | Access/Edge `[VERIFY]`       | Yellow `[VERIFY]`   |

> IS-IS provides the underlay routing for VXLAN VTEP reachability across the Nexus fabric.

---

## Overlay Protocol — BGP EVPN

- **Address Family:** L2VPN EVPN
- **Encapsulation (Core/Agg):** VXLAN
- **VTEP Source:** Loopback interfaces (100.64.0.x/32)
- **Route Types:** Type-2 (MAC/IP), Type-5 (IP Prefix) `[VERIFY]`

### BGP Design

| Parameter        | Value                              |
|-----------------|------------------------------------|
| BGP AS          | `[VERIFY]` from diagram            |
| Route Reflector | core-01, core-02 `[VERIFY]`       |
| EVPN Peers      | Loopback-based iBGP `[VERIFY]`    |

---

## Access Layer

### PON Devices (pon-01 through pon-04)

- **Role:** Passive Optical Network — fiber-based customer access
- **Platform:** Cisco IOSv
- **Protocol:** BGP topology for EVPN route exchange
- **Count:** 4 devices

### TWR Devices (twr-01 through twr-03)

- **Role:** Tower aggregation — wireless/tower-based customer access
- **Platform:** Cisco IOSv
- **Protocol:** OSPF topology for underlay routing
- **Count:** 3 devices

### Interconnection with Core/Aggregation

- **Uplinks:** Access devices connect to core-01/core-02 via point-to-point links
- **IP addressing:** `[VERIFY]` transit link IPs from diagram

---

## VXLAN Configuration (Nexus 9000v)

| Parameter        | Value                   |
|-----------------|-------------------------|
| VNI Range       | `[VERIFY]` from diagram |
| VLAN-to-VNI Map | `[VERIFY]` from diagram |
| Flood Mode      | BGP-based (EVPN)       |
| ARP Suppression | `[VERIFY]`             |
| NVE Interface   | nve1                   |

---

## CML Deployment

| Parameter     | Value                          |
|--------------|--------------------------------|
| CML Instance | https://REDACTED         |
| Lab Name     | isp-network-deployment         |
| Node Images  | Nexus 9000v, IOSv              |

---

## Diagram Reference

See: [Network Image.png](./Network%20Image.png)

---

## Notes

- All values marked `[VERIFY]` need confirmation against the high-resolution topology diagram
- Interface names: Ethernet for Nexus 9000v, GigabitEthernet for IOSv
- Edge: edge-01, edge-02 (IOSv)
- Core/Agg: core-01, core-02, agg-01, agg-02 (Nexus 9000v)
- Access: pon-01–04, twr-01–03 (IOSv)
- Customer cloud connections exist on both the left (agg) and right (access) edges
